# AGENTS.md

This project follows Spec-Driven Development on GitHub, a light convention. The full rules live in [CLAUDE.md](CLAUDE.md) and apply to every agent, including Codex, Cursor, and Copilot.

Summary:

- One spec per epic in `specs/<number>-<name>.md`, such as `specs/001-user-authentication.md`. This is the only artifact you author by hand.
- GitHub issues hold the plan and the state. Epics are parent issues, tasks are sub-issues, order is set with blocking edges.
- The loop: write the spec, derive sub-issues in plan mode, materialize to GitHub in one pass, work in unblocked order, log non-obvious choices in `DECISIONS/<number>-<name>.md`.
- A spec describes the outcome. Choose the method at build time and keep it out of the spec.

Read [CLAUDE.md](CLAUDE.md) for the spec template, the materialize commands, blocking discipline, and the ready-work query.
