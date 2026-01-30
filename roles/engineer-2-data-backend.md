# A backend + data exercise: WhatsApp-group chatbot

This is a small exercise in backend engineering and data systems design.  
The goal is to evaluate how well you understand a fuzzy product requirement, scope it, design a robust system, prototype the core, and communicate tradeoffs clearly.

## Expected deliverables and evaluation criteria
- **Working prototype** that demonstrates the end-to-end flow (ingest → decide to respond → generate → reply).
- **System design quality**: architecture, data modeling.
- **Data thinking**: context retrieval strategy, storage, indexing, evaluation, and observability for model outputs.
- **Communication cadence**: frequent progress updates (PRs/commits + short notes) over a last-day dump.
- **Attention to details**: edge cases,testability.

---

## Problem statement
Design a chatbot that can be **added to WhatsApp groups** (or a realistic workaround) such that:

- People can **mention** the bot (e.g., `@bot`) in a WhatsApp group.
- The bot **answers questions** when mentioned.
- The bot uses **group context** (recent messages, prior bot answers, group “memory”) to respond.
- The bot can **browse/surf the internet** (via a search API + page fetch) when needed, and cite sources.

### Constraints / reality check (intentional)
It might not be possible (or straightforward) to add a bot to a WhatsApp group using official means in your setup. You must propose a **workable integration plan**. 

### Reference implementation (optional)
You **may** use `clawdbot` as a starting point/reference to speed up integration work:  
https://github.com/clawdbot/clawdbot  

However, you are **not required** to use it—you're free to evaluate and use other libraries. 

**Your job is to define a plan, list assumptions, and deliver a runnable prototype.**

---

# Task 1 — Requirements, assumptions, and scope

Create a doc with clear requirements, assumptions, and a minimal but realistic spec.

---

# Task 2 — Ingestion and WhatsApp integration adapter
Build a WhatsApp ingestion service that accepts real WhatsApp group past messages when added in a group, stores them with required fields, detects bot mentions to trigger replies.

---

# Task 3 — Data model for group context + retrieval
Implement a context store and retrieval layer.

---

# Task 4 — The answering pipeline (LLM + grounding + web browsing)
Implement an LLM-driven answering pipeline that routes mentioned queries, assembles group context, optionally performs web search + page fetch for grounding with citations, generates a safe response, and logs/stores prompts, sources, and outputs end-to-end.

### Prototype acceptance criteria
- Given a group conversation, when a user mentions the bot:
  - It responds with an answer grounded in group context
  - If it browses, it includes citations and stores the sources used

---

# Task 5 — Evaluation: prove your bot is not making things up
We care about correctness and trust.

Implement a lightweight evaluation harness:
- A small dataset of 15–30 test prompts (some require browsing, some require group context, some should be refused/clarified).

---

## Repository requirements
- Do not fork. Work on a private repo and add collaborators: `shashank-srikant`, `dipesh517`.
- Use **Python**.
- Add `.gitignore`.
- Keep `README.md` updated with reproducible instructions.

---
**All the best!**
