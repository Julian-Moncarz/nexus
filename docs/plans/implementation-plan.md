# Nexus Implementation Plan

## Overview

Nexus is a web-based chat interface where users ask natural language questions about structured data and receive professional data analyst-quality reports with prose, tables, and charts.

## Architecture

```
┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐
│     Chainlit     │ ───▶ │       E2B        │ ───▶ │    Supabase      │
│     (Chat UI)    │      │  (Agent Runtime) │      │    (Postgres)    │
└──────────────────┘      └──────────────────┘      └──────────────────┘
         │                        │
         ▼                        ▼
┌──────────────────┐      ┌──────────────────┐
│     Langfuse     │      │   PDF Reports    │
│    (Logging)     │      │   (matplotlib)   │
└──────────────────┘      └──────────────────┘
```

## Stack

| Layer | Tool | Purpose |
|-------|------|---------|
| **UI** | Chainlit | Python-based chat interface with streaming, PDF display |
| **Agent Runtime** | E2B | Sandboxed Python execution with internet access |
| **LLM** | Claude (Sonnet) | Reasoning, code generation, report writing |
| **Database** | Supabase | Postgres with Airtable sync capability |
| **Logging** | Langfuse | Full transcript logging and debugging |

## Critical: Heterogeneous Database Structure

**The database is NOT a clean, normalized schema.** It contains copies of tables from different clubs, each with their own naming conventions and structures.

### What the database looks like

```
NOT THIS (clean):                    ACTUALLY THIS (messy):
─────────────────                    ─────────────────────
participants                         berkeley_members
programs                             berkeley_programs
clubs                                mit_fellows
                                     mit_cohorts
                                     stanford_applicants
                                     stanford_programmes
                                     oxford_participants
                                     oxford_reading_groups
                                     ... etc
```

### Example schema variations

| Club | Members Table | "Major" Column | "Stage" Column |
|------|---------------|----------------|----------------|
| Berkeley | `berkeley_members` | `major` | `current_status` |
| MIT | `mit_fellows` | `field_of_study` | `fellowship_stage` |
| Stanford | `stanford_applicants` | `department` | `application_status` |
| Oxford | `oxford_participants` | `programme_of_study` | `participant_level` |

### Why this matters

1. **Agent must discover schema dynamically** — cannot hardcode table/column names
2. **Agent is a data wrangler** — must explore, map, and combine heterogeneous sources
3. **This is why we need E2B** — a static query tool can't handle this; an agent that can explore and adapt can

### Agent workflow for heterogeneous data

```
User: "What's the FTE rate by major across all clubs?"

Agent:
1. Discover tables:
   SELECT table_name FROM information_schema.tables WHERE table_schema = 'public'

2. Explore schemas:
   SELECT column_name FROM information_schema.columns WHERE table_name = 'berkeley_members'
   ... repeat for each table

3. Map semantics:
   - "major" ≈ "field_of_study" ≈ "department" ≈ "programme_of_study"
   - "current_status" ≈ "fellowship_stage" ≈ "participant_level"

4. Write unifying query/transformation:
   dfs = []
   for table, config in table_mappings.items():
       df = pd.read_sql(f"SELECT {config['major_col']}, {config['stage_col']} FROM {table}", conn)
       df.columns = ['major', 'stage']  # normalize
       df['club'] = config['club_name']
       dfs.append(df)
   combined = pd.concat(dfs)

5. Analyze combined data
6. Generate report
```

## How This Satisfies the PRD

### Landing State
> *"Clean chat interface, title, subtitle, suggested starter questions"*

- Chainlit provides a clean default UI
- Configure title ("Nexus") and subtitle in Chainlit settings
- Starter questions rendered as clickable buttons via `cl.Action`

### Asking a Question
> *"User types naturally in chat input"*

- Chainlit's `cl.on_message` handler receives user input
- No constraints on question format — natural language

### Working State
> *"Working... indicator, expandable trace panel showing thinking, tool calls, intermediate results"*

- Chainlit's `cl.Step` context manager shows nested execution steps
- Stream agent thinking via `cl.Message.stream_token()`
- Tool calls (SQL queries, Python execution) displayed as collapsible steps
- Langfuse captures full transcript for debugging

### Report Output
> *"Prose summary, tables, charts, source note"*

The agent runs in E2B and:
1. Queries Supabase via `psycopg2`
2. Analyzes data with `pandas`
3. Generates charts with `matplotlib`
4. Compiles a PDF report with prose + charts + tables
5. Returns PDF to Chainlit for display

Chainlit renders the PDF inline with download option.

### Follow-up Questions
> *"Conversation context maintained"*

- Chainlit maintains session state automatically
- Agent receives conversation history for context
- One note here is that the actually same E2B instance should be kept running for at least a bit - that way any python scripts or decisions or things are saved?

### Visual Design
> *"Clean/minimal, white background, sans-serif, Linear/Notion aesthetic"*

- Chainlit's default theme is clean and minimal
- Custom CSS can be added via `.chainlit/config.toml`

### Data Architecture
> *"Data-agnostic: swapping the dataset requires no code changes"*

Configuration-driven approach:
```python
# config.py
DATASET_CONFIG = {
    "title": "Nexus",
    "subtitle": "Ask questions across club data",
    "db_url": os.environ["SUPABASE_URL"],
    "suggested_questions": [
        "What's the correlation between major and becoming FTE?",
        "What's the completion rate for intro fellowships?",
    ]
}
```

To swap datasets: change `db_url` and `suggested_questions`. No code changes.

## Implementation Strategy (Revised)

### Key Decisions
1. **E2B Lifecycle:** One sandbox per Chainlit session, persists across follow-up messages, killed on `@cl.on_chat_end`
2. **Agent Orchestration:** Claude API runs reasoning loop (not sandboxed). Chainlit calls Claude, Claude calls tool, loop continues.
3. **Tool Pattern:** `execute_python(code: str, sandbox_id: str)` → Claude sends code, gets stdout/stderr, repeats
4. **Output:** PDF generated in E2B by Claude, returned as base64, displayed via `cl.File()`
5. **Data Safety:** Large results truncated (df.head(50)), env vars passed to E2B, no creds in code
6. **Model:** Claude Sonnet (reasoning sufficient for schema discovery + analysis)
7. **Streaming:** Show intermediate traces (schema discovery, queries) via `cl.Step()`, final PDF batch-displayed
8. **No E2B Template:** Bare Python 3.11 sandbox, pip install on first message

### Phase 1: Chainlit + E2B Integration
1. Set up Supabase with mock heterogeneous data (3 clubs, 5-10 columns per table, ~100 rows each)
2. Create Chainlit app with:
   - `@cl.on_message` handler that gets/creates session E2B sandbox
   - `execute_python` tool definition (code + sandbox_id → execution)
   - `@cl.on_chat_end` hook to kill sandbox
3. Test E2B sandbox creation, persistence, cleanup

### Phase 2: Agent Loop
1. Draft Claude system prompt: explore schema → map semantics → analyze → generate PDF
2. Implement message handler:
   - Create/reuse session E2B sandbox
   - Call Claude with tools + conversation history
   - Stream `cl.Step()` for each tool call
   - Truncate large outputs (df.head(50))
   - Continue until Claude returns final PDF
3. Test with 1-2 example questions

### Phase 3: UI Polish
1. Chainlit config (title, subtitle, suggested questions)
2. Custom CSS for clean aesthetic
3. Error handling + user messages
4. PDF display verification

### Phase 4: Deployment Ready
1. Environment variables documented
2. Supabase project setup guide
3. Deploy to Fly.io (or local for spike)

## File Structure

```
nexus/
├── app.py                 # Chainlit app entry point
├── agent.py               # E2B + Claude agent logic
├── config.py              # Dataset configuration
├── prompts/
│   └── system.md          # Agent system prompt
├── tools/
│   └── sandbox.py         # E2B sandbox utilities
├── scripts/
│   └── seed_database.py   # Mock data seeder
├── .chainlit/
│   └── config.toml        # Chainlit UI config
└── requirements.txt
```

## Environment Variables

```bash
ANTHROPIC_API_KEY=sk-ant-...
E2B_API_KEY=e2b_...
SUPABASE_URL=postgresql://...
LANGFUSE_PUBLIC_KEY=pk-...
LANGFUSE_SECRET_KEY=sk-...
```

## Example Flow

**User:** "What's the correlation between major and becoming FTE?"

**Agent (in E2B):**

**Step 1: Discover schema**
```python
import psycopg2, pandas as pd

conn = psycopg2.connect(os.environ["SUPABASE_URL"])

# What tables exist?
tables = pd.read_sql("""
    SELECT table_name FROM information_schema.tables
    WHERE table_schema = 'public'
""", conn)
print(tables)
# Output: berkeley_members, mit_fellows, stanford_applicants, oxford_participants, ...
```

**Step 2: Explore each table's columns**
```python
for table in ['berkeley_members', 'mit_fellows', 'stanford_applicants']:
    cols = pd.read_sql(f"""
        SELECT column_name FROM information_schema.columns
        WHERE table_name = '{table}'
    """, conn)
    print(f"{table}: {cols['column_name'].tolist()}")

# Output:
# berkeley_members: ['id', 'name', 'major', 'current_status', ...]
# mit_fellows: ['id', 'name', 'field_of_study', 'fellowship_stage', ...]
# stanford_applicants: ['id', 'full_name', 'department', 'application_status', ...]
```

**Step 3: Map semantics and combine**
```python
table_configs = {
    'berkeley_members': {'major': 'major', 'stage': 'current_status', 'club': 'Berkeley'},
    'mit_fellows': {'major': 'field_of_study', 'stage': 'fellowship_stage', 'club': 'MIT'},
    'stanford_applicants': {'major': 'department', 'stage': 'application_status', 'club': 'Stanford'},
}

dfs = []
for table, cfg in table_configs.items():
    df = pd.read_sql(f"SELECT {cfg['major']} as major, {cfg['stage']} as stage FROM {table}", conn)
    df['club'] = cfg['club']
    dfs.append(df)

combined = pd.concat(dfs, ignore_index=True)
```

**Step 4: Analyze and generate chart**
```python
import matplotlib.pyplot as plt

# Normalize stage values (different clubs use different terms for FTE)
fte_terms = ['fte', 'full_time', 'employed', 'staff']
combined['is_fte'] = combined['stage'].str.lower().isin(fte_terms)

fte_rate = combined.groupby('major')['is_fte'].mean().sort_values(ascending=False)

plt.figure(figsize=(10, 6))
fte_rate.plot(kind='bar')
plt.title('FTE Rate by Major (Across All Clubs)')
plt.ylabel('FTE Rate')
plt.tight_layout()
plt.savefig('chart.png')
```

**Step 5: Generate PDF report and return**

**Chainlit:** Displays PDF inline with "Here's your analysis" message.

**Langfuse:** Logs full transcript (question → schema discovery → data wrangling → analysis → report).

## Success Criteria

From PRD:
> *"Neav opens a deployed URL, asks a question, watches the agent work, and receives a polished report with prose, tables, and charts. He immediately understands the vision."*

This implementation achieves that by:
- Clean, professional chat UI (Chainlit)
- Visible agent "working" state with tool call traces
- PDF reports with prose, charts, and tables (matplotlib + reportlab)
- Easy deployment (Fly.io)
- Data-agnostic config (swap Supabase connection string)
