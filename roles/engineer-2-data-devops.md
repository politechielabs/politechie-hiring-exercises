# A data + devops exercise: Wikipedia live-edits pipeline

A small exercise in data engineering and infrastructure.
The goal is to evaluate how well you understand a fuzzy product requirement, scope it, design a robust data pipeline, run it on real infrastructure, and communicate tradeoffs clearly.

**Time budget: ~4 days.** If you find yourself in scope creep, stop and write down what you cut.

## Expected deliverables and evaluation criteria

- **Working prototype** that ingests a live public event stream, models it into analytics tables, and serves them, end-to-end.
- **System design quality**: clean separation between ingestion, storage, transformation and serving; sensible failure handling.
- **Data correctness**: no gaps, no duplicates, idempotent reprocessing, and correct handling of late or out-of-order events.
- **Scalability & production readiness**: aim for a deliberate design, one that holds up as event volume, history and the number of consumers grow.
- **Infrastructure as code & automated delivery**: the infrastructure is reproducible from the repo and deploys happen through CI/CD, not by hand.
- **Operability**: someone other than you can tell whether the pipeline is healthy, and what to do when it isn't, without SSHing into the box.
- **Deployment**: the pipeline actually runs on the public internet, not just on your laptop.
- **Communication cadence**: frequent progress updates (PRs/commits + short notes) over a last-day dump.
- **Attention to details**: edge cases, retries, testability.
- **Design decisions documented**: a `design-decisions.md` capturing the key choices you made, *why*, and the tradeoffs you knowingly accepted (what you optimized for, what you gave up).
- **Forward thinking**: the README articulates how the system would evolve — known limitations, future advancements, and what it would take to run this in production.
- **Clarifications**: reach out if you have any doubts about the deliverables or implementation. Don’t just assume or sit on it.

---

## Problem statement

A small analytics team wants near-real-time and historical insight into how Wikipedia is being edited. Nobody on the team wants to babysit a pipeline, so it has to run on its own, recover on its own, and tell someone when it can't.

The source is Wikimedia's public **EventStreams** feed — a Server-Sent Events (SSE) stream of every change across all Wikimedia wikis, free and with no authentication:

```
https://stream.wikimedia.org/v2/stream/recentchange
```

Things worth knowing about the source (read the [EventStreams docs](https://wikitech.wikimedia.org/wiki/Event_Platform/EventStreams_HTTP_Service) before designing):

- It is a real, continuous stream — expect tens of events per second, all day.
- Each event carries a unique `meta.id`, an event time (`meta.dt` / `timestamp`) and a `bot` flag, among other fields.
- The stream can be resumed from a previous position (`Last-Event-ID`) and replayed from a point in time (`since`), within a limited retention window.
- The server closes connections periodically; your client will be disconnected routinely, not just on failure.
- Events can arrive late or out of order relative to their event time.
- Wikimedia asks clients to send a descriptive `User-Agent`.

The team needs the following data products:

| Question                                               | Grain       |
|--------------------------------------------------------|-------------|
| Edits per wiki, split bot vs human                     | Hourly      |
| Top 10 most-edited pages per wiki                      | Rolling 24h |
| Distinct active editors per wiki, split bot vs human   | Daily       |

You may deliberately scope down (e.g. to a subset of wikis or event types) — if you do, say what you cut and why. Assume your deployment target is a small, cheap host with limited disk; your design should account for that.

### Constraints

- **Python** for application code. **Postgres** as the warehouse.
- **Idempotency is mandatory.** Restarting the ingester, replaying the stream, or re-running a transformation over the same window must never double-count.
- **Freshness.** The data products should be no more than **15 minutes** behind the stream under normal operation.
- **Test-driven.** Use `pytest`. Transformations need tests too, not just Python code.

---

## Task 1 — Ingestion

Build a long-running consumer for the stream that lands raw events durably.

- It must survive crashes and redeploys: on restart it resumes from where it left off, without gaps and without duplicates.
- Malformed or unexpected events must not crash the consumer or be silently dropped. Decide where they go.
- Document the delivery semantics you guarantee and how you achieve them.

## Task 2 — Modelling & transformation

Turn raw events into the three data products above.

- Layer the models however you see fit (e.g. raw → staging → marts). Plain SQL or dbt — your choice, justify it.
- Process incrementally. Late-arriving events must correctly update windows that were already computed.
- Provide a `backfill` command that takes a time range. Running it twice over the same range must produce identical results.
- Add **data-quality checks** (e.g. uniqueness, nulls, freshness, volume anomalies) that fail loudly when violated.
- Define a storage strategy: partitioning, indexing and retention for raw and modelled data, given the disk constraint above.

## Task 3 — Orchestration & observability

- Schedule and orchestrate the transformations and checks. Airflow, Dagster, Prefect, cron, or something else — justify the choice for a pipeline of this size.
- Expose pipeline metrics (at minimum: ingest rate, ingest lag, last successful transformation run, data-quality failures, database size) in a form you can graph — e.g. Prometheus + Grafana or equivalent.
- Define **at least one real alert** (e.g. "ingestion stalled for > 5 minutes") that fires somewhere a human would see it.
- Write a short `RUNBOOK.md`: for each alert, what it means, how to confirm it, and how to recover.

## Task 4 — Infrastructure & deployment

Package the pipeline and actually deploy it.

- `Dockerfile`(s) and a `docker-compose.yml` such that `docker compose up` on a clean clone brings up the full stack with data flowing.
- **Infrastructure as code.** Provision your cloud resources with Terraform (or an equivalent IaC tool). Secrets must not live in the repo.
- **Schema migrations** are versioned and applied automatically as part of deployment.
- **GitHub Actions CI/CD.** CI runs lint, unit tests, and transformation/SQL tests against a real Postgres (e.g. a service container) on every push and PR. CD builds and pushes images to a registry (Docker Hub or GHCR) and deploys to your host on every merge to `main`.
- **Live deployment.** The pipeline must be running on a publicly reachable host with a `/health` endpoint, plus a public read-only view of the data products (a dashboard or a small API — your choice). Host it for free if you can. If free hosting is a blocker, let us know and we'll provide you with a VM to deploy on.
- README must include the public `/health` URL and the URL of the data products view.

---

## Acceptance criteria

A reviewer should be able to:

1. Visit the public `/health` URL and get `200 OK`, then open the public dashboard/API and see data products no more than 15 minutes old.
2. Clone the repo, follow only the README, run `docker compose up`, and see the data products populated within a few minutes.
3. Stop the ingester for 10+ minutes, restart it, and verify — using queries provided in the README — that raw data has no gap and no duplicates over that period.
4. Run `backfill` over a past hour twice and get identical data-product rows both times.
5. Follow README steps to force a failure (e.g. stall ingestion or break a data-quality check) and see it surface as a failed run and a fired alert.
6. Read the IaC and understand exactly what is provisioned; `apply` / `destroy` steps are documented, and the deployed infrastructure matches the code.
7. See green CI on PRs and at least one automated deploy from `main` in the GitHub Actions history.
8. Run the test suite locally.

---
## Engineering reasoning (write this down — we grade it)

Code shows what you built; we also want to see how you think and design.

### In `design-decisions.md`
- **Design choices + tradeoffs.** For each significant decision, state the choice, the alternatives you considered, and the tradeoff you accepted. We are explicitly looking for *conscious* tradeoffs, not justifications after the fact.
- Call out the components you designed for a production-level setup but didn't build due to *time constraints*, if any, and any parts of the design you know don't scale.

### In `README.md`
- **Further advancements.** If you had another week/month, what would you build or harden next, and why those first? Think beyond features — correctness, scale, observability, developer experience.
- **Path to production.** See the section below — this one is required.

### Production readiness (required — do not skip)

This is a prototype. We care a lot about whether you can see the gap between "works on a demo" and "runs in production." In the README, include a **"From prototype to production"** section that honestly answers:

- What in this codebase would you *not* trust in production as-is, and why?
- What breaks first at 10× and 100× the event volume, or with a year of history, and how would you address it? Where does Postgres stop being the right warehouse?
- Concretely walk through: secrets management, storage growth and retention, reliability and recovery (restarts, partial failures, poison events, source outages longer than its retention window), backfills at scale, schema evolution of the source, monitoring and alerting (what would you page on?), backups and disaster recovery, data migrations, security, and cost.

A short, specific list of *real* gaps beats a long generic checklist.

---

## Repository requirements

- Do not fork. Work on a private repo and add collaborators: `shashank-srikant`, `dipesh517`, `prateekb1912`
- Use **Python**. Postgres for databases.
- Keep `README.md` updated with reproducible instructions.
- Include a `design-decisions.md` and a `RUNBOOK.md` at the repo root. Keep `design-decisions.md` updated as you go — we'd rather see decisions recorded in the moment than reconstructed at the end.
- Keep track of all prompts (claude/copilot) in a version-controlled file `prompts.md` so that the evolution of the prompts is tracked over time.
- Commits should reflect incremental progress — we read the history.

---

**All the best!**
