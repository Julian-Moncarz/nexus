# Nexus Demo PRD

## Overview
Nexus is a web-based chat interface where users ask natural language questions about structured data and receive professional data analyst-quality reports. For the demo, it uses mock AI safety club data, but the system is data-agnostic.

## Success Criteria
Neav opens a deployed URL, asks a question, watches the agent work, and receives a polished report with prose, tables, and charts. He immediately understands the vision. The only step to production is swapping mock data for real data.

## User Experience

### Landing State
- Clean chat interface
- Title: "Nexus"
- Subtitle: "Ask questions across club data" (configurable per dataset)
- 3-4 suggested starter questions as clickable chips (configurable per dataset)

### Asking a Question
- User types naturally in chat input
- Sends message

### Working State
- "Working..." indicator appears
- Expandable trace panel showing full execution:
  - Thinking blocks
  - Tool calls (data queries, Python execution)
  - Intermediate results
- User can watch the agent work in real-time

### Report Output
- Appears complete (not streamed)
- Contains:
  - **Prose summary:** Direct, data-analyst tone. Key numbers bolded.
  - **Tables:** Clean, readable. Proper headers and alignment.
  - **Charts:** Bar, line, or scatter as appropriate. Professional styling.
  - **Source note:** "Based on data from X clubs, Y participants" (or equivalent for dataset)

### Follow-up Questions
- Conversation context maintained
- User can drill down or ask related questions

## Visual Design
- Style: Clean/minimal (white background, sans-serif, Linear/Notion aesthetic)
- Responsive: Works on desktop, acceptable on tablet
- Dark mode: Not required for demo

## Data Architecture
The system connects to a structured dataset and executes queries/analysis against it.

**For demo:** Mock AI safety club data (see `schema.md`)

**For production:** Real data synced from club Airtables

**Data-agnostic requirement:** Swapping the underlying dataset (and updating suggested questions) should require no code changes to the chat/agent layer.

## Example Queries (Demo)
1. "Correlation between major and becoming FTE"
2. "What's the completion rate for intro fellowships?"
3. "Which program format has best retention?"
4. "Do people with higher value alignment stick around longer?"

## Out of Scope for Demo
- User authentication
- Multi-tenant data isolation
- Real data sync from Airtables
- Conversation persistence across sessions
