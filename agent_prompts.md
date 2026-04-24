# Agent Prompts

This document describes the role, inputs, outputs, and prompt design for each GPT-4o agent in the AI Learning Agent system. There are 6 agents across 2 workflows.

Prompts are implemented as system messages in n8n's OpenAI node. Dynamic values (user profile fields, topic weights, article content) are injected at runtime via n8n expressions.

---

## Workflow 1 Agents

### 1. Orchestrator Agent

**Role:** Supervisor and planner. Reads the user profile and topic weights, decides which topics to prioritise for this run, builds a search brief for the Scout, and generates a run manifest to track agent health across the full workflow.

**Inputs:**
- Full user profile (name, role, topics interested, preferred sources, learning streak)
- All 13 topic weights (engagement count, avg quiz score, last seen date, weight)

**Output (JSON):**
```json
{
  "search_brief": {
    "primary_topics": ["...", "..."],
    "secondary_topics": ["...", "..."],
    "preferred_sources": ["...", "..."],
    "avoid_topics": ["..."],
    "content_type": "...",
    "reasoning": "..."
  },
  "run_manifest": {
    "expected_article_count": 2,
    "min_title_length": 10,
    "min_summary_length": 50,
    "require_quiz_questions": true,
    "require_correct_answers": true,
    "retry_budget": 1,
    "abort_on_gate_failure": true
  }
}
```

**Prompt design notes:**
- Told to weight topics with low avg_quiz_score and high engagement count more heavily
- Told to avoid topics seen in the last 3 days unless they are high priority
- Told to prefer practical, product-focused content over academic papers
- The run manifest sets quality thresholds that Quality Gates 1, 2, and 3 enforce downstream

---

### 2. Scout Agent

**Role:** Content finder. Takes the search brief from the Orchestrator and retrieves candidate articles from multiple live sources.

**Sources searched (real-time RSS and HTTP calls):**
- HackerNews (top stories API)
- arXiv (AI/ML recent papers)
- Lenny's Newsletter (RSS)
- Stratechery (RSS)
- The Batch by DeepLearning.AI (RSS)

**Inputs:**
- Orchestrator search brief (primary topics, secondary topics, preferred sources, content type)

**Output:** A list of candidate articles, each with title, URL, source, and a short description of why it matches the brief.

**Prompt design notes:**
- Told to return at least 4 to 6 candidates so the Ranker has options
- Told to prefer content published within the last 7 days
- Told to flag if a source returned no results so the Orchestrator can retry
- Explicitly told to avoid academic papers unless the search brief requests them

---

### 3. Ranker Agent

**Role:** Evaluator. Takes candidate articles from Scout and scores each one on relevance, novelty, and signal quality. Selects the top 2 to deliver.

**Inputs:**
- List of candidate articles from Scout
- User profile (role, target role, topics interested)
- List of previously delivered article URLs (to prevent duplicates)

**Output (JSON):**
```json
{
  "selected_articles": [
    {
      "title": "...",
      "url": "...",
      "source": "...",
      "relevance_score": 8,
      "novelty_score": 9,
      "signal_quality_score": 8,
      "topic_tags": ["RAG", "LLM evaluation frameworks"],
      "why_selected": "..."
    }
  ],
  "rejected_articles": [
    {
      "title": "...",
      "url": "...",
      "reason_rejected": "..."
    }
  ]
}
```

**Prompt design notes:**
- Told to score relevance against the user's current role and growth goal, not just topic match
- Told to penalise articles that are too theoretical or too introductory for the user's level
- Told to reject any URL already present in the articles table (duplicate check)
- Told to always return exactly 2 selected articles; if fewer than 2 quality articles exist, fail the gate rather than send weak content

---

### 4. Explainer + Quiz Agent

**Role:** Teacher. Takes the 2 selected articles and generates a plain-English explanation of each, a "why it matters" section, and 2 quiz questions with model answers.

**Inputs:**
- Title, URL, source, and topic tags for each of the 2 selected articles
- User profile (role, level, target role)

**Output (JSON):**
```json
{
  "articles": [
    {
      "title": "...",
      "summary": "...",
      "why_it_matters": "...",
      "quiz_questions": ["...", "..."],
      "correct_answers": ["...", "..."]
    }
  ]
}
```

**Prompt design notes:**
- Told to write summaries at a beginner-to-intermediate level; avoid jargon without explanation
- Told to make "why it matters" specific to an AI PM role, not generic
- Told to write quiz questions that test understanding, not recall; answers should require the user to apply the concept, not just repeat it
- Told to keep each summary between 100 and 150 words; quiz questions should be a single clear sentence

---

## Workflow 2 Agents

### 5. Scorer Agent

**Role:** Assessor. Evaluates the user's quiz reply against the model answers and returns a structured score with diagnostic detail.

**Inputs:**
- User's raw email reply text
- Original quiz questions
- Model answers

**Output (JSON):**
```json
{
  "understanding_level": 7,
  "topics_understood": ["RAG", "Vector databases"],
  "topics_to_reinforce": ["Inference optimization"],
  "difficulty_adjustment": "maintain",
  "feedback": "Good grasp of retrieval mechanics. The answer on inference missed the latency vs cost tradeoff — worth revisiting."
}
```

**Prompt design notes:**
- Told to accept free-form replies; penalise based on conceptual gaps, not phrasing or formatting
- Told to be calibrated: a score of 7 means solid understanding with one clear gap, not a polite pass
- understanding_level is 0 to 10; below 5 triggers heavier weighting of that topic in future digests
- difficulty_adjustment options are "increase", "maintain", or "decrease"
- feedback should be 1 to 2 sentences, specific, and actionable

---

### 6. Topic Normaliser Agent

**Role:** Mapper. Takes the free-form topic names returned by the Scorer (topics_understood, topics_to_reinforce) and maps them to exact canonical topic names before Airtable is updated.

**Why this exists:** GPT-4o often returns topic names in slightly different forms, e.g. "retrieval augmented generation" instead of "RAG", or "agent orchestration" instead of "AI agents and orchestration". A direct Airtable lookup on a non-matching string would fail silently. This agent enforces exact match.

**Inputs:**
- Free-form topic name from Scorer output
- The full list of 13 canonical topic names

**Output:** A single canonical topic name string, exactly as it appears in the topic_weights table.

**Prompt design notes:**
- Told to return only the canonical name, no explanation or preamble
- Told that if no match can be found with high confidence, return "UNKNOWN" so the workflow can skip the update rather than writing a bad value
- This is a short, focused call; max tokens kept low

---

## Notes on prompt versioning

Prompts live inside n8n nodes and are not currently version controlled separately. If you modify a prompt, note the change in CHANGELOG.md with the date and what was changed. The biggest risk is prompt drift: small edits to one agent can break the structured JSON contract that downstream agents depend on.
