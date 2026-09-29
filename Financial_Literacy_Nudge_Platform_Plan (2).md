# Digital Financial Literacy Nudge Platform for College Students
### Project Plan — Agentic/RAG Automation using n8n

---

## a. The Use Case (Refined)

**"FinBuddy" — a behavior-based financial nudging agent for college students.**

Instead of a generic budgeting app, the innovative angle is:
- Students connect a (sandboxed) bank account or upload spending data.
- An **AI agent watches spending patterns** (not just totals — patterns like "spends 40% of budget in first 5 days," "impulse food delivery spikes on weekends").
- It runs those patterns through a **RAG-powered knowledge base of financial literacy content** (articles, rules of thumb, university financial-aid rules, etc.) to generate **personalized, bite-sized nudges** ("You've spent ₹1,200 on food delivery this week — that's 3x your average. Here's a 60-second read on the 50/30/20 rule.")
- Nudges are gamified: streaks, XP, badges, weekly "budget challenges" (e.g., "No-spend weekend"), and a leaderboard among opted-in peer groups (hostel/class).
- A simulated investment sandbox lets them practice with fake money based on real market data, tying literacy lessons to action.

This is a good fit for n8n because the whole thing is really an **event-driven pipeline**: data comes in → gets analyzed by an LLM agent → retrieves relevant content (RAG) → triggers a personalized action (notification/gamification update) → logs outcome for the next cycle.

---

## b. Overall Architecture

```
┌─────────────────┐     ┌──────────────────────────────────────────┐
│  Data Sources    │     │              n8n (Orchestrator)            │
│  - Plaid Sandbox │────▶│  Workflow 1: Ingest & Categorize Txns      │
│  - CSV upload    │     │  Workflow 2: Pattern Detection Agent        │
│  - Manual entry  │     │  Workflow 3: RAG Nudge Generator            │
└─────────────────┘     │  Workflow 4: Gamification Engine            │
                          │  Workflow 5: Notification Dispatcher       │
                          │  Workflow 6: Weekly Report/Dashboard Sync  │
                          └───────────────┬────────────────────────────┘
                                          │
        ┌─────────────────────────────────┼─────────────────────────────┐
        ▼                                 ▼                             ▼
┌───────────────┐               ┌──────────────────┐          ┌──────────────────┐
│ Postgres/       │               │ Vector DB          │          │ LLM (OpenAI/      │
│ Supabase        │               │ (Pinecone/         │          │ Claude API)        │
│ (users, txns,   │               │ Supabase pgvector) │          │ - categorization   │
│ points, badges) │               │ - financial        │          │ - nudge copy       │
└───────────────┘               │   literacy corpus  │          │ - chatbot Q&A       │
                                  └──────────────────┘          └──────────────────┘
        │
        ▼
┌──────────────────┐    ┌─────────────────┐    ┌───────────────────┐
│ Notification layer │    │ Frontend/Dashboard│    │ Analytics/         │
│ (Twilio WhatsApp,  │    │ (React/Next.js or │    │ Reporting          │
│  SendGrid, OneSignal│    │  low-code: Glide/ │    │ (Metabase/Retool)  │
│  push)             │    │  Softr on Supabase)│    │                    │
└──────────────────┘    └─────────────────┘    └───────────────────┘
```

**Flow in one line:** Transactions → categorized by LLM → pattern flags raised → RAG retrieves relevant literacy content → LLM composes personalized nudge → gamification engine scores it → nudge delivered via WhatsApp/push → dashboard updates → loop weekly.

---

## c. Implementation Components

| Component | Purpose | Suggested Tool |
|---|---|---|
| **Orchestration layer** | Wires every workflow together, schedules, webhooks | **n8n** (self-hosted via Docker, or n8n Cloud) |
| **Relational DB** | Users, transactions, budgets, points, badges, challenge state | **Supabase (Postgres)** or Firebase |
| **Vector DB** | Store embedded financial literacy articles/snippets for RAG | **Supabase pgvector** (simplest, same DB) or Pinecone/Weaviate |
| **LLM** | Categorize transactions, detect patterns in natural language, write nudges, power chatbot | **OpenAI GPT-4o-mini** or **Claude (Haiku/Sonnet via API)** — cheap model for categorization, stronger model for nudge generation |
| **Embeddings model** | Convert literacy articles + user queries into vectors | OpenAI `text-embedding-3-small` or open-source `bge-small-en` |
| **Bank data source** | Realistic transaction data without real banking integration risk | **Plaid Sandbox API** (fake but realistic) or a Python/Faker synthetic data generator feeding n8n via webhook |
| **Notification/Nudge delivery** | Push the actual nudge to the student | **Twilio WhatsApp API**, **SendGrid** (email), or **OneSignal** (push notifications) |
| **Frontend/Dashboard** | Student-facing app: budget view, badges, leaderboard, chat | **Next.js/React** (built via vibe coding) or no-code **Glide/Softr** wired to Supabase |
| **Gamification logic** | Points, streaks, badges, challenges | Custom logic inside n8n workflow + Supabase tables (no separate tool needed) |
| **Analytics/Reporting** | Track "measurable improvement" in habits — the core success metric | **Metabase** or **Retool** dashboards on top of Supabase |
| **Auth** | Student login/signup | Supabase Auth or Firebase Auth |

---

## d. Technologies/Techniques to Study

- **n8n basics**: Webhook node, Schedule Trigger, HTTP Request node, Code node (JS), Postgres/Supabase node, IF/Switch nodes, Merge node, Loop Over Items, Error workflows.
- **n8n AI nodes**: AI Agent node, OpenAI/Anthropic Chat Model node, Embeddings node, Vector Store node (built-in Pinecone/Supabase/Qdrant integration), Memory node (for chatbot conversation context).
- **RAG technique**: chunking strategy for literacy articles (~300–500 tokens/chunk), embedding, similarity search (top-k retrieval), prompt-stuffing pattern (retrieved context + user spending summary → LLM).
- **Prompt engineering**: structured prompts for categorization (constrain to a fixed category list), for nudge tone (empathetic, non-judgmental, short).
- **Plaid Sandbox**: how test users/transactions work, `/transactions/get` endpoint.
- **Behavioral nudging theory** (optional but strengthens the "why"): loss aversion, streaks/variable rewards, social proof (leaderboards) — useful to cite in your report as the psychological basis for the gamification design.
- **Webhooks & polling**: how n8n triggers workflows on schedule vs. on event (useful for "nudge only when a pattern is detected" logic).

---

## e. APIs Likely Needed

1. **Plaid API** (Sandbox tier, free) — transaction data simulation
2. **OpenAI API** or **Anthropic API** — categorization, nudge generation, RAG synthesis, chatbot
3. **OpenAI Embeddings API** (or open-source local embedding model if avoiding cost)
4. **Supabase API** (Postgres + pgvector + Auth, all one project)
5. **Twilio API** (WhatsApp/SMS) — free trial credits available
6. **SendGrid API** — email nudges (has a generous free tier)
7. **OneSignal API** — push notifications (free tier)
8. Optional: **Alpha Vantage / Yahoo Finance API** — free market data to power the "simulated investment challenge"

---

## f. Implementation Approach (Vibe Coding + Open Source + n8n)

**Step 1 — Environment setup**
- Spin up n8n via Docker (`docker run n8n.io/n8n`) or use n8n Cloud free trial.
- Create a Supabase project → enable `pgvector` extension → create tables: `users`, `transactions`, `budgets`, `nudges`, `points`, `badges`, `challenges`.

**Step 2 — Seed the knowledge base (RAG)**
- Collect ~30–50 short financial literacy articles/snippets (budgeting rules, saving tips, credit basics — can be AI-generated then human-reviewed, or pulled from open sources like NISM/RBI financial literacy content).
- Build an n8n workflow: read articles → chunk → embed → store in Supabase pgvector.

**Step 3 — Data ingestion workflow**
- n8n Schedule Trigger (daily) → HTTP Request to Plaid Sandbox `/transactions/get` → Code node to clean data → LLM node to categorize each transaction → insert into Supabase `transactions` table.

**Step 4 — Pattern detection + RAG nudge workflow**
- Trigger after ingestion completes → Code node computes simple stats (spend vs. average, category spikes) → AI Agent node: given the stats + retrieved vector-store context → generate a short, personalized nudge (2–3 sentences + one actionable tip).
- Store nudge in `nudges` table.

**Step 5 — Gamification workflow**
- Whenever a student stays under budget / completes a challenge → n8n calculates points, checks badge thresholds, updates `points`/`badges` tables.

**Step 6 — Delivery workflow**
- New nudge/badge → Twilio/SendGrid/OneSignal node → sends it to the student.

**Step 7 — Frontend**
- Use Claude/Cursor ("vibe coding") to scaffold a Next.js dashboard reading from Supabase: shows budget vs. actual, badges, leaderboard, and a chat box (calls an n8n webhook that runs a RAG-powered financial Q&A agent).
- Alternatively, no-code it fast with Glide/Softr directly on Supabase tables if time is short.

**Step 8 — Testing**
- Use Plaid Sandbox test users + synthetic edge cases (overspending, sudden income, etc.) to validate that nudges trigger correctly and gamification logic behaves.

---

## g. Deployment & Marketing

**Deployment**
- n8n: n8n Cloud (easiest) or self-host on Railway/Render (free/cheap tier).
- Supabase: hosted free tier.
- Frontend: deploy on Vercel (free tier, one-click from GitHub).
- Keep all API keys in n8n credentials store (never hard-coded).

**Marketing (for a "measurable improvement" pilot)**
- Pilot with a campus finance/economics club or hostel group (20–50 students) — small enough to hand-hold, large enough for data.
- Track a **baseline week** (no nudges) vs. **nudge-active weeks** — compare savings rate / budget adherence as your "measurable improvement" metric for the report.
- Use Instagram/WhatsApp campus groups + a referral badge ("invite 3 friends, unlock a badge") for organic growth.
- Partner angle: pitch to the college's student welfare office or a fintech-for-students startup as a low-cost engagement tool.

---

## Suggested Report/Demo Structure
1. Problem framing (financial illiteracy stats among students)
2. Architecture diagram (above)
3. Live n8n workflow walkthrough (screenshot each workflow)
4. RAG demo: show a query retrieving the right literacy snippet
5. Gamification demo: badge/points unlocking
6. Pilot results: baseline vs. nudge-week spending behavior
7. Future scope: real bank integration, group challenges, financial-aid literacy modules

---

## Alternative n8n Project Ideas (if you want options)
1. **Campus Mental Health Check-in Bot** — RAG over university counseling resources + WhatsApp check-ins + escalation workflow for at-risk responses.
2. **AI Research Paper Digest Agent** — pulls arXiv papers on a topic weekly, RAG-summarizes, emails a digest; agentic layer answers follow-up questions.
3. **Smart Job/Internship Matcher** — scrapes listings, embeds them, matches against a student's resume via vector similarity, sends curated matches.
4. **Local Business Review-to-Insight Agent** — pulls Google/Zomato reviews, RAG-summarizes sentiment trends, generates an owner-facing action digest.
5. **Automated Scholarship Finder** — agent that crawls scholarship databases, filters by eligibility (RAG over eligibility docs), and nudges students before deadlines.

The financial literacy idea you already have is strong and well-suited to n8n's strengths (multi-step automation + AI agent + RAG) — I'd recommend sticking with it rather than switching.
