# Future Plans

## ARIA — Agentic Memory & Safety Layer for mySerenity's Mental Health Chatbot

**Status:** Planned (not started) — captured 2026-07-23
**Why this project:** Real, running proof of agentic AI systems design — memory architecture, model portability, and safety-critical escalation — built on top of the existing mySerenity platform rather than a toy demo. Directly supports the "Applied AI Systems Engineer" positioning: backend/systems fluency (FastAPI, Postgres, Docker/Kubernetes already in place) combined with genuine agentic AI judgment, not just framework familiarity.

**Context:** mySerenity already has an AI chatbot, ARIA, intended to chat with users who may be mental health patients. Chat history is already persisted in a database. The goal is to add agentic memory (vector DB over chat summaries) and a dual model backend, at zero/near-zero cost, as a portfolio-grade showcase project.

---

### 1. Model layer — dual backend, swappable from day one

- **Local:** Ollama running Qwen2.5 (7B or 14B depending on hardware) — free, full control over inference, demonstrates the ability to run and operate model infrastructure directly rather than only calling a hosted API.
- **Hosted fallback:** OpenRouter free-tier models — useful both as a fallback if local inference is too slow/weak on nuanced conversational turns, and as a natural A/B comparison point (local vs. hosted quality) that's itself worth writing up.
- **Design decision:** build one interface abstraction over both backends from the start, not local-first-then-migrate. Small extra abstraction now, pays off as a concrete "how would you handle model portability" interview story later.

### 2. Where the vector DB fits — cross-session memory only

- Existing Postgres chat history stays as the raw, complete record. The vector DB (Qdrant or Chroma — lightweight, free, runs locally alongside Ollama) is a derived, lossy-but-searchable index on top of it, not a replacement.
- Flow: after a session ends (or periodically during a long one), a summarization step condenses the conversation into a short factual summary, embeds it, and stores it in the vector DB.
- At the start of a new session, or when the agent decides it needs prior context, embed the current conversation's gist and retrieve the most relevant past summaries — same retrieval mechanism as classic RAG over documents, applied to the user's own history instead.
- Explicitly NOT for within-session context — that stays in the normal context window as usual.

### 3. The actual agentic decision point (this is what makes it "agentic," not just "RAG with steps")

- Naive version: always pull past summaries into every response. Wasteful, and risks surfacing old context the user didn't want revisited in this moment.
- Better version: an orchestrator (or lightweight classifier) decides, per turn, whether this message plausibly benefits from history — e.g. "I've been feeling anxious again" probably does, "what's a good breathing exercise" probably doesn't.
- This per-turn retrieval decision is the component most likely to come up in a staff-level interview follow-up ("why does it retrieve here and not there") — worth being able to defend concretely because it was actually built, not rehearsed.

### 4. Safety and escalation — first-class, non-negotiable component (not an afterthought)

Given the domain (users who may be mental health patients), this cannot be bolted on after the demo works — same posture as treating auth or idempotency as non-negotiable in a payments system.

- A lightweight classifier or keyword-plus-LLM check runs on **every incoming message**, specifically for crisis signals (self-harm, suicidal ideation, immediate danger).
- On a crisis signal, route to a **fixed, non-negotiable safe response** (crisis resources, encouragement to contact a real professional) — this response is NOT generated freely by the model in that moment.
- This gate sits **outside** the normal agentic reasoning loop entirely — a hard check that runs before the model's own response is ever shown to the user, never something the model itself is trusted to decide on the fly.
- System prompt must be explicit that ARIA is a supportive conversational tool, not a therapist or diagnostician — nudge toward professional care, never attempt to treat or diagnose.
- This is both responsible design and a strong interview differentiator — few candidates building agent demos can speak concretely to a safety-critical escalation path they actually built.

### 5. Suggested build order

1. Dual-backend model layer (Ollama + OpenRouter fallback) working against existing chat history and Postgres store — least risky, working baseline fast.
2. Safety/crisis-detection gate — before anything agentic, must work even in the simplest version of the system.
3. Summarization job + vector DB retrieval — the part that makes it agentic; least urgent to get exactly right on day one.

### 6. Evaluation

- Runs through all three build stages, not just added at the end.
- For this domain specifically: the eval set needs deliberate crisis-language test cases to confirm the safety gate actually fires — not just general conversation quality checks.

---

## How to use this doc

Add new future project plans below as separate `##` sections, dated, following the same structure: status, why, then a numbered technical plan. Keep completed/abandoned plans here too (marked as such) rather than deleting, so the reasoning trail persists.
