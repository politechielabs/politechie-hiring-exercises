# A backend exercise: Discord → JIRA sync bot

A small exercise in backend engineering and systems integration.
The goal is to evaluate how well you understand a fuzzy product requirement, scope it, design a robust system, prototype the core, and communicate tradeoffs clearly.

**Time budget: ~2.5 days.** If you find yourself in scope creep, stop and write down what you cut.

## Expected deliverables and evaluation criteria

- **Working prototype** that mirrors Discord project activity into a real JIRA Cloud instance end-to-end.
- **System design quality**: clean module boundaries, idempotent sync, sensible failure handling.
- **Deployment**: the bot actually runs on the public internet, not just on your laptop.
- **Communication cadence**: frequent progress updates (PRs/commits + short notes) over a last-day dump.
- **Attention to details**: edge cases, retries, testability.
- **Design decisions documented**: a `design-decisions.md` capturing the key choices you made, *why*, and the tradeoffs you knowingly accepted (what you optimized for, what you gave up).
- **Forward thinking**: the README articulates how the system would evolve — known limitations, future advancements, and what it would take to run this in production.

---

## Problem statement

Consider a small engineering team that operates **Discord-first** and has no project manager. Work happens in Discord, but reporting is expected in JIRA, and nobody wants to double-write. Build a bot that bridges the two.

Assume the following Discord convention:

- Channels prefixed `proj-` are **project channels** (e.g. `proj-checkout`).
- **Threads** under a project channel are **milestones** of that project.
- Discord users map to JIRA accounts via a simple config file you decide the shape of.

Entity mapping the bot should maintain:

| Discord                          | JIRA                                |
|----------------------------------|-------------------------------------|
| `proj-<name>` channel            | JIRA project, key `<NAME>`          |
| Thread under a project channel   | JIRA issue (type: Story)            |
| Message in that thread           | Comment, optionally a status transition |

JIRA project creation is out of scope — if the project doesn't exist, the bot logs and skips.

### Supported actions (v1 — full list)

1. **Create issue.** New thread under a `proj-*` channel → create a Story in the matching JIRA project, with thread author as reporter.
2. **Comment.** New message in a milestone thread → comment on the linked issue, attributed to the message author.
3. **Transition.** Message in a milestone thread containing `status: <todo|in_progress|done|blocked>` → transition the linked issue. The comment is still recorded.
4. **Resync command.** `!resync` in any project channel → ensure every thread has a corresponding JIRA issue; create missing, never delete; reply in-channel with a summary.


### Constraints

- **JIRA Cloud.** Set up your own free Atlassian sandbox.
- **Idempotency is mandatory.** Re-running the bot over the same Discord history, or restarting after a crash, must not produce duplicate JIRA mutations.
- **Test-driven.** Use `pytest`.

---

## Task 1 — JIRA client

Build a thin Python wrapper around the JIRA Cloud REST API for the operations the bot needs (lookup, create issue, comment, transition). Design the interface; document how you achieve idempotency for repeat calls.

## Task 2 — Discord ingestion

The bot should backfill channel and thread history on startup, listen to live events, and reliably hand them to the sync engine — including surviving a crash with unprocessed events queued up.

## Task 3 — Deployment

Package the bot as a Docker image and actually deploy it.

- `Dockerfile` and `docker-compose.yml` for local runs.
- **GitHub Actions CI/CD.** CI runs lint and tests on every push and PR. CD builds and pushes the image to **Docker Hub** on every merge to `main`.
- **Live deployment.** The bot must be running on a publicly reachable host with a `/health` endpoint. Host it for free if you can. If free hosting is a blocker, let us know and we'll provide you with a VM to deploy on.
- README must include the public `/health` URL, an invite link to your test Discord server.

---

## Acceptance criteria

A reviewer should be able to:

1. Visit the public `/health` URL and get `200 OK`. Join your test Discord server, create a thread under a `proj-*` channel, and watch the corresponding Story appear in your JIRA sandbox (screenshots or a short recording of the JIRA side are fine if the sandbox isn't publicly shareable).
2. Pull the published Docker Hub image, follow the README, and run a second instance end-to-end on their own JIRA + Discord.
3. Post `wrapping this up, status: done` in a milestone thread and see the issue transition to `Done` with the message as a comment.
4. Run `!resync` and see a summary message with no duplicate issues created.
5. Stop the deployed bot, post several messages, restart, and see all of them reflected in JIRA exactly once.
6. Run the test suite locally.

---
## Engineering reasoning (write this down — we grade it)

Code shows what you built; we also want to see how you think.

### In `design-decisions.md`
- **Design choices + tradeoffs.** For each significant decision, state the choice, the alternatives you considered, and the tradeoff you accepted. We are explicitly looking for *conscious* tradeoffs, not justifications after the fact.
- Be honest about what you'd do differently with more time.

### In `README.md`
- **Further advancements.** If you had another week/month, what would you build or harden next, and why those first? Think beyond features — correctness, scale, observability, developer experience.
- **Path to production.** See the section below — this one is required.

### Production readiness (required — do not skip)

This is a prototype. We care a lot about whether you can see the gap between "works on a demo" and "runs in production." In the README, include a **"From prototype to production"** section that honestly answers:

- What in this codebase would you *not* trust in production as-is, and why?
- What breaks first under real load or real failure, and how would you address it?
- Concretely walk through: secrets management, rate limits, reliability and recovery (restarts, partial failures), monitoring and alerting (what would you page on?), data migrations, security, and cost.

A short, specific list of *real* gaps beats a long generic checklist.

---

## Repository requirements

- Do not fork. Work on a private repo and add collaborators: `shashank-srikant`, `dipesh517`.
- Use **Python**. Postgres for databases.
- Keep `README.md` updated with reproducible instructions.
- Include a `design-decisions.md` at the repo root. Keep it updated as you go — we'd rather see decisions recorded in the moment than reconstructed at the end.
- Commits should reflect incremental progress — we read the history.

---

**All the best!**
