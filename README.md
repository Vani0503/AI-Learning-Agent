═══════════════════════════════════════════════════════════
PROJECT: AI Learning Agent — Personal AI PM Knowledge OS
═══════════════════════════════════════════════════════════

WHAT IT IS
An autonomous multi-agent AI system that acts as a personal
AI learning operating system for an AI PM. It finds relevant
content daily, explains it in PM terms, tests retention,
tracks a structured learning curriculum, benchmarks against
industry peers, evaluates its own performance, and adapts
its behavior based on how I interact with it over time.

WHY I BUILT IT
Targeting FAANG AI PM roles in NYC. Built this to stay
genuinely current on the AI landscape AND to get deep
hands-on experience with: agentic AI, multi-agent
orchestration, MCP tool use, n8n, evals, and memory
architecture — the exact stack I need to understand
as a product leader in this space.

WHAT MAKES IT AGENTIC (not just automation)
→ Orchestrator reads memory before deciding what to search
→ Scout uses MCP tools autonomously mid-reasoning
→ Agents observe user behavior and adapt tomorrow's output
→ Human-in-loop: Intent Agent asks what to learn today
→ Feedback loop: quiz scores change future content selection
→ Self-evaluation: system measures its own output quality

────────────────────────────────────────────────────────────
AGENTS (7 total)
────────────────────────────────────────────────────────────
1. Intent Agent     → asks what to learn today (6:45am)
                      reads your reply via webhook
2. Orchestrator     → reads memory, plans the day,
                      delegates to all sub-agents, adapts
3. Scout            → finds content via MCP tools
                      (arXiv, HN, RSS, web scrape)
4. Ranker           → scores + shortlists top 3 articles
                      using memory + topic weights
5. Explainer        → writes PM-focused summaries
6. Quiz             → generates + scores retention questions
7. Curriculum Agent → weekly: updates mastered/in-progress/
                      backlog, sends learning report,
                      benchmarks against AI PM peer level

────────────────────────────────────────────────────────────
MEMORY ARCHITECTURE — Airtable (6 tables)
────────────────────────────────────────────────────────────
user_profile     → topics, preferences, scores, streak
articles         → content fetched, scored, summarized
interactions     → every read/skip/quiz action logged
knowledge_store  → structured record of what was learned
curriculum       → mastered / in_progress / backlog buckets
topic_weights    → per-topic engagement + quiz performance

────────────────────────────────────────────────────────────
EVALS PIPELINE (separate workflow)
────────────────────────────────────────────────────────────
→ Scout relevance score  (user rates articles 1–5)
→ Explainer quality      (Claude-as-judge vs rubric)
→ Quiz calibration       (score distribution over time)
→ Adaptation quality     (week 1 vs week 4 behavior delta)

────────────────────────────────────────────────────────────
DELIVERY
────────────────────────────────────────────────────────────
Daily 6:45am  → Gmail: "What do you want to learn today?"
Daily 7:15am  → Gmail: full digest (top 3 articles +
                summaries + quiz questions)
Weekly Sunday → Gmail: Learning Report (curriculum update
                + peer benchmark + eval scores)

────────────────────────────────────────────────────────────
TECH STACK
────────────────────────────────────────────────────────────
Orchestration  : n8n (cloud)
AI / Agents    : OpenAI GPT-4o with tool use
Memory         : Airtable (6-table schema)
Delivery       : Gmail (via n8n OAuth)
Data sources   : [TO FILL — you'll tell us]
MCP Tools      : arXiv API, HackerNews API, RSS fetch,
                 web scraper, Airtable R/W, Gmail send

────────────────────────────────────────────────────────────
BUILD PHASES
────────────────────────────────────────────────────────────
Phase 1 (now)  : n8n + OpenAI + Airtable
Phase 2        : Rebuild in Python + OpenAI SDK
Phase 3        : Claude Code CLI
Phase 4        : Full MCP server + advanced memory

────────────────────────────────────────────────────────────
WHAT I LEARNED BUILDING THIS
────────────────────────────────────────────────────────────
[fill after completion — for README + LinkedIn]
═══════════════════════════════════════════════════════════

Execution:
n8n link  http://localhost:5678

Product Decisions Log — AI Learning Agent
PD-001 — Memory backend: Airtable over a database
Chose Airtable for the memory layer because it gives visual transparency into what the agent is storing and thinking. A PM should be able to see and edit the agent's memory without engineering help. Transparency over performance at this stage.
PD-002 — Six tables, not one
Separated concerns deliberately — behavior (interactions) is stored separately from knowledge (knowledge_store) because they answer different questions. Behavior tells you what the user did. Knowledge tells you what the user understood. Conflating them would make querying messy and agent prompts less precise.
PD-003 — Seeding curriculum manually
The initial topic backlog was PM-curated, not AI-generated. This is intentional — the PM defines the learning goal, the agent executes toward it. Fully delegating curriculum design to AI removes accountability for what the user actually learns.
PD-004 — topic_weights as a separate table
Instead of computing relevance scores on the fly, we pre-compute and store weights per topic. This makes the Ranker faster and more transparent — you can open Airtable and see exactly why certain topics are being prioritized.
PD-005 — skip_count as a first-class signal
Explicit negative signal (skipping) is tracked separately from positive signal (reading). Most recommendation systems underweight negative signals. We treat skips as equally important to reads.
PD-007 — Memory package as a single clean JSON object
Instead of passing raw Airtable rows to the Orchestrator, we format memory into a structured package first. This makes the Orchestrator prompt cleaner, reduces token usage, and makes the system easier to debug. Always transform data before passing it to an LLM

Product Decision to log
PD-006 — Token scopes as a PM consideration
When designing integrations, PMs need to specify exact permission scopes required — not just "connect to Airtable." Under-scoped tokens cause silent failures. Scope requirements should be part of the technical spec.
