# AI Learning Agent

A personalized daily AI learning digest with a quiz feedback loop that adapts to how well you're actually learning, built on n8n, GPT-4o, and Airtable.

---

## What this is

Most learning tools give you the same content regardless of how you're doing. This one doesn't.

Every morning, it pulls articles from across the AI ecosystem, explains them at your level, and sends you a quiz. When you reply with your answers, it scores your understanding, figures out which concepts you're weak on, and quietly adjusts what it sends you tomorrow. Over time, it builds a picture of where you're strong and where you're not, and it changes the content mix accordingly.

It was built for one specific person, a PM upskilling in the ever-evolving landscape of LLMs, AI, and ML, across 13 topics in AI product management. Everything about the system, the sources it pulls from, the way it explains things, the topics it covers, is configured around that goal.

---

## The problem it solves

There's no shortage of AI content. The problem is that none of it knows who you are, what you already know, or what you actually need to work on next.

If you're a PM trying to get serious about AI, you're probably doing one of the following: subscribing to newsletters you skim and forget; taking courses that don't adapt when you're bored or when you're lost; saving articles to a read-later folder that never gets read; or using quiz apps that test isolated facts with no connection to your actual job.

This system replaces all of that with a single daily email that gets smarter the longer you use it. It knows which topics you're avoiding. It knows which ones you've already got. It adjusts without you having to manage it.

---

## Why we built it this way

**n8n for orchestration, not logic.** n8n is the backbone; it handles scheduling, Gmail polling, Airtable reads/writes, and wiring agents together. But the actual thinking (what topics to prioritise, how to rank articles, how to score a quiz reply) lives in GPT-4o. This keeps the system flexible: the logic can be upgraded by changing a prompt, not rewriting a workflow.

**GPT-4o as the reasoning layer.** Every agent in this system is a GPT-4o call with a specific role and a specific output format. The Orchestrator plans. The Scout finds. The Ranker evaluates. The Explainer teaches. The Scorer assesses. Each one is isolated, which means each one can be improved independently.

**Airtable as the memory layer.** Topic weights, quiz scores, article history, learning progress; all of it lives in Airtable. This was a deliberate choice over a database because it makes the system inspectable. You can open a table and see exactly why the agent decided to send you RAG content three days in a row. There's no black box.

**Email as the interface.** No app to open, no dashboard to check. The digest arrives in your inbox every morning. Your quiz reply is just a normal email reply. This was a deliberate UX decision; the lowest-friction interface possible so the habit actually sticks.

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   WORKFLOW 1 — Daily Digest          │
│                                                     │
│  Schedule Trigger (daily)                           │
│       │                                             │
│       ▼                                             │
│  Airtable Read ──► user_profile + topic_weights     │
│       │                                             │
│       ▼                                             │
│  Orchestrator Agent (GPT-4o)                        │
│  Builds search brief. Prioritises topics by weight. │
│  Generates run manifest to track agent health.      │
│       │                                             │
│       ▼                                             │
│  Quality Gate 1 ──── FAIL ──► Failure Email         │
│       │                                             │
│       ▼                                             │
│  Scout Agent (GPT-4o)                               │
│  Searches HackerNews, arXiv, The Batch,             │
│  Stratechery, Lenny's Newsletter (live RSS).        │
│  Returns candidate articles.                        │
│       │                                             │
│       ▼                                             │
│  Quality Gate 2 ──── FAIL ──► Failure Email         │
│       │                                             │
│       ▼                                             │
│  Ranker Agent (GPT-4o)                              │
│  Scores articles on relevance, novelty, signal.     │
│  Selects top 2. Writes to articles table.           │
│       │                                             │
│       ▼                                             │
│  Quality Gate 3 ──── FAIL ──► Failure Email         │
│       │                                             │
│       ▼                                             │
│  Explainer + Quiz Agent (GPT-4o)                    │
│  Writes plain-English summary + 2 quiz questions    │
│  with model answers per article.                    │
│       │                                             │
│       ▼                                             │
│  Gmail Send ──► HTML digest email to user           │
│       │                                             │
│       ▼                                             │
│  Airtable Write ──► interactions table              │
│  (thread_id, quiz_questions, correct_answers)       │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│              WORKFLOW 2 — Reply Feedback Loop        │
│                                                     │
│  Gmail Trigger (polls hourly)                       │
│       │                                             │
│       ▼                                             │
│  Extract reply text + thread_id                     │
│       │                                             │
│       ▼                                             │
│  Airtable Lookup ──► fetch quiz from interactions   │
│       │                                             │
│       ▼                                             │
│  Scorer Agent (GPT-4o)                              │
│  Compares reply vs model answers.                   │
│  Returns understanding_level (0-10), topics         │
│  understood, topics to reinforce, feedback.         │
│       │                                             │
│       ▼                                             │
│  Topic Normaliser Agent (GPT-4o)                    │
│  Maps free-form topic names to canonical list.      │
│  Ensures exact match before Airtable update.        │
│       │                                             │
│       ▼                                             │
│  Airtable Updates:                                  │
│  ├── interactions: quiz_score, notes                │
│  ├── topic_weights: engagement_count, avg_quiz_score│
│  └── knowledge_store: concept_learned, confidence  │
└─────────────────────────────────────────────────────┘
```

**Stack:** n8n (hosted on Railway) for scheduling, Gmail, Airtable, and agent wiring; GPT-4o for all reasoning across orchestration, search, ranking, explanation, and scoring; Airtable for persistent memory across topic weights, quiz history, articles, and learning progress; Gmail for delivery and reply capture.

---

## How it works day to day

**Morning: the digest arrives**

You get an email with two articles. Each one has a plain-English explanation and two quiz questions at the bottom. The articles were selected that morning based on which topics you're weakest on, which you haven't seen recently, and what your historical engagement looks like. End-to-end, Workflow 1 completes in roughly 25 to 30 seconds, makes approximately 5,500 tokens of GPT-4o calls across all agents, and costs under $0.03 per run; around $0.80 per month at daily cadence.

**During the day: you reply**

You reply to the email with your answers. It can be a sentence, a paragraph, a brain dump; the scorer handles free-form responses. You don't need to format anything.

**Within the hour: it's scored**

Workflow 2 picks up your reply, compares it against the model answers, and returns a score from 0 to 10 along with which concepts you understood and which need reinforcement. This takes roughly 6 to 8 seconds and updates your topic weights in Airtable automatically.

**Tomorrow: the mix shifts**

If you scored well on RAG but struggled with inference optimisation, tomorrow's digest will weight toward inference. If you haven't seen a topic in two weeks, it'll resurface. The adjustment is gradual and automatic; you don't configure anything.

---

## Airtable schema

Six tables, each with a specific role in the system.

**interactions** — One record per digest sent. Stores the thread_id that links a Gmail reply back to the correct quiz, the quiz questions and model answers as JSON, and the quiz score once the user replies.

**topic_weights** — One record per topic across the 13 canonical topics. Tracks engagement count, average quiz score, skip count, last seen date, and a composite weight that drives Orchestrator decisions.

**user_profile** — Static user context: role, target role, preferred sources, topics of interest. Fed into the Orchestrator prompt at the start of every Workflow 1 run.

**articles** — Every article fetched, ranked, and delivered. Prevents duplicates. Tracks relevance score, novelty score, summary, and topic tags per article.

**knowledge_store** — Concept-level learning log. Written to after every scored reply. Stores what concept was learned, confidence score, which article taught it, and retention status over time.

**curriculum** — Structured learning plan across all 13 topics. Tracks priority, current level vs peer benchmark, and target completion dates. Intended to be updated automatically as quiz scores improve.

**The 13 canonical topics:**
AI agents and orchestration · LLM evaluation frameworks · RAG · Multi-agent systems · Prompt engineering · Fine-tuning vs prompting · Vector databases · Multimodal product design · AI product metrics · Inference optimization · AI safety and alignment · LLM cost optimization · AI regulatory landscape

---

## Current limitations

**Topic suggestion is manual.** There's no agent yet that analyses quiz performance and suggests new topics to add. If your interests evolve or the field moves on, you update the topic list yourself.

**Voice reply isn't supported.** You have to type your quiz answers as an email reply. A voice-to-text path (via Whisper API) was designed but not built; partly because the input channel (email can't natively accept audio) needs a UX decision first.

**Industry curriculum alignment is static.** The 13 topics were defined once at setup. There's no agent that monitors what AI PMs at top tech companies are actually expected to know right now and adjusts the curriculum accordingly. That agent is designed but not built.

---

## Setup

**Prerequisites**

n8n instance (local or hosted; Railway recommended), an OpenAI API key with GPT-4o access, an Airtable account with base configured per the schema above, and a Gmail account with OAuth credentials.

**Steps**
1. Clone this repo
2. Copy `.env.example` to `.env` and fill in your credentials
3. Import both workflow JSONs into n8n
4. Set up your Airtable base with the 6 tables and fields described above
5. Populate `user_profile` with your details and `topic_weights` with starting weights for each of the 13 topics
6. Activate Workflow 1; it will run on its daily schedule
7. Activate Workflow 2; it will poll Gmail hourly for replies

**Credentials needed**
```
OPENAI_API_KEY=
AIRTABLE_API_KEY=
AIRTABLE_BASE_ID=
GMAIL_OAUTH_CREDENTIALS=
```

---

## What's next

The system is stable and running 24/7. Features on the backlog, roughly in priority order: richer content sourcing (more RSS feeds, direct URL browsing for Scout); token, cost, and latency tracking per run; curriculum table connected to workflow (auto-updates as scores improve); topic suggestion agent (analyses performance, proposes new topics for approval); industry curriculum alignment agent (monitors industry expectations, suggests updates); voice reply via Whisper API.

---

## About

Built by Vani, a PM with 6 years of experience currently pursuing an MBA at Cornell. Vani is passionate about building user-centric products that leverage technology, and this project sits at the intersection of that: a learning system that actually adapts to the person using it, built with AI tooling as both the subject and the medium.

Questions, feedback, or want to build something similar? Reach out on LinkedIn.
