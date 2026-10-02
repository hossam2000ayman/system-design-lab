# CLAUDE.md — Context for AI coding agents

> Copy this file as `AGENTS.md` too, so other AI tools read the same rules.

## What this repo is
A personal **learning lab** for system design. Each folder in `labs/` is an
experiment that tests a hypothesis written in `practice.md`. The goal is
**understanding**, not shipping features fast.

## Who I am
A software engineer leveling up from beginner to advanced in system design.
Explain trade-offs, not just solutions.

## Rules for the agent
1. **Read first.** Before changing anything in `labs/NN-*`, read the matching lab section in `practice.md` (hypothesis, setup, steps).
2. **Don't design for me.** If I ask a design question, ask me clarifying questions and critique my proposal. Only give a full solution if I explicitly write `REVEAL`.
3. **Scaffold, explain, stay minimal.** When generating infra or code, comment every non-obvious line, and don't add components I didn't ask for.
4. **Pin versions.** Use pinned Docker image tags and dependency versions. Never add a new dependency without telling me its name, purpose, and why it is needed, so I can verify it exists and is trustworthy.
5. **Never cheat tests.** Don't delete, skip, or weaken tests, assertions, or performance thresholds to make something pass. Tell me it fails and why.
6. **Small changes.** Keep each change small and reviewable; summarize what changed and how to verify it.
7. **Numbers go in RESULTS.md.** Don't invent or estimate results. Only record what was actually measured.
8. **No secrets in code.** Use `.env` files (gitignored) and example files like `.env.example`.

## Conventions
- Each lab: `labs/NN-short-name/` with `docker-compose.yml`, `README.md` (how to run), `RESULTS.md`.
- Load tests: k6 scripts named `loadtest.js`.
- Decisions: `decisions/ADR-XXX-title.md` using the template in the root `README.md`.

## Common commands (per lab)
```bash
docker compose up -d          # start the lab
k6 run loadtest.js            # run the load test
docker compose logs -f <svc>  # inspect a service
docker compose down -v        # stop and delete volumes (fresh start)
```
