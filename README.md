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
