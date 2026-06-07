# A backend exercise: Discord → JIRA sync bot

A small exercise in backend engineering and API integration.
The goal is to see how well you can read a spec, integrate two external services, write clean Python, and back it up with tests.

**Time budget: ~2 days.** If you find yourself in scope creep, stop and write down what you cut.

## Expected deliverables and evaluation criteria

- **Working prototype** that mirrors Discord activity into a JIRA Cloud instance.
- **Code quality**: clear module boundaries, readable Python.
- **Tests**: meaningful unit tests for the JIRA client and the sync logic.
- **Communication**: frequent progress updates (commits + short notes) over a last-day dump.

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

### Constraints

- **JIRA Cloud.** Set up your own free Atlassian sandbox.
- **No duplicates.** If the bot sees the same Discord message twice (e.g. on restart), it should not create a duplicate JIRA comment or issue. Keep a small local record of what you've already synced — SQLite is fine.
- **Include tests.** Use `pytest`. At minimum, the JIRA client and the message-parsing logic should be unit tested with mocks (no real network in tests).

---

## Task 1 — JIRA client

Build a thin Python wrapper around the JIRA Cloud REST API for the operations the bot needs (lookup, create issue, comment, transition). Design the interface and document it briefly in the README.

## Task 2 — Discord bot

Build the Discord side using `discord.py` or `py-cord`. The bot should listen for new threads under `proj-*` channels and new messages inside those threads, and call the sync logic for each event.


## Task 3 — Run locally

A reviewer should be able to clone your repo, follow the README, set their env vars (Discord token, JIRA credentials), and run the bot locally with a single command (e.g. `python -m bot` or `make run`).

---

## Acceptance criteria

A reviewer should be able to:

1. Follow the README, set env vars, and start the bot locally.
2. Create a thread under a `proj-*` channel in their test Discord server and see the corresponding Story appear in their JIRA sandbox.
3. Post `wrapping this up, status: done` in that thread and see the issue transition to `Done` with the message as a comment.
4. Stop the bot, post a couple of messages, restart, and see no duplicate JIRA mutations for messages the bot had already processed.
5. Run `pytest` and see it pass.

---

## Repository requirements

- Do not fork. Work on a private repo and add collaborators: `dipesh517`, `shashank-srikant`.
- Use **Python**. SQLite is fine for local state.
- Keep `README.md` updated with reproducible instructions.
- Commits should reflect incremental progress — we read the history.

---

**All the best!**
