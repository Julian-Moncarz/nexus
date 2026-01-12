# Airtable Cross-Club Integration: Options Analysis

[Julian and Neav NDQ call about data sharing](https://www.notion.so/Julian-and-Neav-NDQ-call-about-data-sharing-2dc1a1849c7b80958648d946f33eb029?pvs=21)

[What data to collect](https://www.notion.so/What-data-to-collect-2e31a1849c7b8011b056c9e2b38a45ce?pvs=21)

A while ago I was working on the strategy doc for our group. **I had a few questions that could be answered by data which other groups may have, but we do not** because we are a younger org. Examples:

- what is the conversion rate for people who do some sort of returning fellows / paper reading group program to get into other fellowships eg SPAR?
- What is the completes SPAR → goes on to become FTE in AIS conversion rate?
- What factors (ex major, stated value alignment) are correlated with {insert thing we want ex not dropping out of an intro fellowship}

This gave me an idea:

## **Cross-club data sharing portal**

Clubs could make their data accessible to other clubs. Potentially put the shared data into a simple portal for organizers, where an organizer could ask a question and get an answer quickly from an AI agent that has access to the datasets, has a runtime environment to run analyses and returns a short report with results (also make the raw data available to others

### Example:

**Organizer**: {goes to portal}

“% of people who have completed two or more AI safety fellowships that go on to become full-time in AI safety?”

**Agent**: {Runs analysis} {presents clean report}

Potential big issue: I don't know how many clubs have good data for this to be built on - this is also an opportunity. If we made a guide on what data to collect and how to collect it s.t. it was reasonably easy to get started, I think a lot of clubs would collect the data and thus have much better visibility into the effects of their programming? (Pathfinder could potentially mandate / strongly encourage this + have people use tools that let the data easily sync to a shared portal?) Idea: create and share template Airtable.

https://ais-groups.slack.com/archives/C05MV9Y1B1V/p1767231353581919

## Extension: also create a central repo of club’s Notion / Google Docs

Put an agent on top

**User:** find me all the intro fellowship curriculums anyone uses

**Agent:** Finds them, presents links + recommendation + summary

**Structure**

- Add an upvote system for everyone
- Strong curation system from trusted subset of people who can designate "this is best"
- AI interface

## **Sustainability** (this applies to both projects)

- Open source
- Docs
- Co-maintainers
  - 1+ less commitment-filled co-maintainers who you delegate tasks to with the intention: "I'm going to run this for three months then it's yours"
- KISS
- Catchy name



# what data to collect

## **Track movement**

- {applicant, fellowship participant, regular member, organizer/has done fellowships that are MATS equivish, fte/alumni, no longer engages with our org}
- record this as time series data ({stage} on {date})

## Participant features

- **Free time budget** they have in hours/week (explicit number beats “busy”)
- **Motivation type** (pick one or more?): curiosity, career, altruistic impact, social, prestige
- **Program**
- **Grad / undergrad / PhD**
- **Self-rate value alignment?**
- **Anonymized participant ID** (stable per person) track each applicant in an anonymized way (so you as an organizer at xyz university have access, but not other universities/kairos/wtv)
- **Technical skill:** find a cheap way to measure this - goals:
  - minimize time to complete
  - maximize predictive power for getting into fellowships

## Group info?

For each uni have them give us some basic stats about themselves and their org

- Size of exec team
- num years active
- etc?

## Program description for each program

- Curriculum version (link to syllabus / reading list hash)
- Facilitator identity + experience level
- Cohort size, session length, number of sessions
- Format: discussion vs lecture vs co-working vs other
- Assignments: types + expected hours
- Incentives used: food, certificates, 1:1 mentoring, etc.
- Outreach channels: posters, referrals, class announcements, etc.



# Ways to get data

claude research report

## The Core Challenge

Each club has their own Airtable base → need to aggregate into a central platform → enable AI queries across all data.

------

## Option 1: Native Airtable Multi-Source Sync (SIMPLEST)

How it works: Each club shares a view from their base. Your central "master" base pulls from all of them using https://support.airtable.com/docs/multi-source-syncing.

**Pros**:

- No code required
- Native Airtable feature, reliable
- Real-time-ish sync (automatic on paid plans)
- Can use https://www.airtable.com/platform/ai-agents for queries

**Cons**:

- Requires paid plans for automatic sync (free = manual sync only)
- 10,000 record limit per sync source
- All clubs need to enable "sync outside org" in enterprise settings
- Need consistent field naming across clubs (field mapping required)

Possible idea here: Kairos buys airtable for all clubs, negotiates with them for a cheaper price + support?

------

## Option 2: Airtable API + Custom Script

How it works: Python script (https://pyairtable.readthedocs.io/en/stable/api.html) pulls from each club's base, aggregates into your central database.

Pros:

- Works with free Airtable accounts
- Full control over transformation/normalization
- Can handle schema inconsistencies

Cons:

- Requires coding + hosting (cron job or serverless function)
- Rate limited: 5 requests/second/base
- You need each club's API token (security consideration)
- More maintenance burden

Verdict: Good middle ground if technical resources exist and clubs are on free plans.

------

## Option 3: ETL to PostgreSQL (MOST POWERFUL)

How it works: Use https://airbyte.com/how-to-sync/airtable-to-postgresql-destination (open source, self-hosted) or https://community.airtable.com/development-apis-11/read-write-airtable-with-sql-3898 to sync all Airtables → PostgreSQL. Query with SQL. Connect AI agent to Postgres.

Pros:

- SQL queries across ALL data (most flexible)
- Best for complex analysis
- Airbyte is free/open-source
- Can handle any scale

Cons:

- Requires hosting PostgreSQL + Airbyte
- More complex setup
- Overkill for simple queries

Verdict: Best if you anticipate complex cross-club analysis or large data volumes.

------

## Option 4: n8n/FlowiseAI Agent (BEST AI EXPERIENCE)

How it works: Use https://n8n.io/workflows/2700-ai-agent-to-chat-with-airtable-and-analyze-data/ that connects to Airtable and translates natural language → Airtable queries.

Pros:

- Natural language queries out of the box
- Can connect multiple bases
- Visual workflow builder
- Self-hostable (free)

Cons:

- Queries individual bases, not a unified view
- Still need consistent schemas
- Another tool to maintain

Verdict: Good for the AI interface layer, but still need aggregation underneath.

------

## Claude’s Recommendation: Hybrid Approach

### For OASIS (simplest MVP):

1. Airtable Multi-Source Sync for data aggregation

- Create template base clubs copy
- Each club enables sync sharing
- Central master base pulls all views

1. Airtable's native AI for queries

- Already integrated
- No extra infrastructure
- "Ask AI" button in interface

Cost: ~$20/month for one Team seat on the master base. Clubs can stay on free if they only need to share (not receive) synced data.

### For V2 (if you need more power):

- Add Airbyte → PostgreSQL for complex SQL analysis
- Build custom AI agent with Claude API

------

## Critical Success Factor: Schema Standardization

Whatever option you choose, the biggest work is getting all clubs to use identical field names and types. Create:

1. Template Airtable they copy exactly
2. Data dictionary defining each field
3. Validation view that flags non-compliant data

------

## Sources:

- https://support.airtable.com/docs/multi-source-syncing
- https://support.airtable.com/docs/getting-started-with-airtable-sync
- https://pyairtable.readthedocs.io/en/stable/api.html
- https://airbyte.com/how-to-sync/airtable-to-postgresql-destination
- https://n8n.io/workflows/2700-ai-agent-to-chat-with-airtable-and-analyze-data/
- https://www.airtable.com/platform/ai-agents