# Spec-Driven Development on GitHub

A single convention for running spec-driven development on GitHub's native issue features. One short spec per epic. The issue graph carries the breakdown, the order, and the state.

## The Problem

Spec-driven development tools keep growing a project-management layer of their own. Specs, task lists, status, and dependencies all land in files: a `specs/` tree, a state JSON, steering documents. That layer made sense when GitHub issues were flat and could not express hierarchy or dependencies.

Three problems follow from carrying it:

- **A second source of truth.** Status lives in files and also in the issue tracker, and the two drift. The code says one thing, the state file another, the open issues a third.
- **No home for dependencies.** Each spec sits alone in its folder. The everyday statement "this epic needs that one first" has nowhere native to live, so the order survives in a teammate's memory.
- **State that dies on reset.** A plan held in chat or scratch files vanishes when the context resets, and the agent rebuilds it from scratch, often differently.

## The Solution

GitHub issues now express hierarchy and dependencies directly, all scriptable through `gh`: sub-issues, blocking relationships, and issue types. The project-management layer already exists, in the place the team already looks. Four principles put it to work.

| Principle | Addresses |
| --- | --- |
| **Spec First** | Scattered intent, specs that drift from the code |
| **Graph Over Files** | A second source of truth, state lost on reset |
| **Fence With Non-Goals** | Scope creep, bloated epics |
| **Derive, Then Work the Ready Set** | Hand-written task lists, unclear order |

## What It Looks Like

Take a feature: user accounts. Start with a one-page spec, `specs/auth.md`:

```
# User Authentication — Spec
Status: Draft

## Why
Today everything is anonymous and lost on refresh. Users need accounts to save work.

## What
Email-and-password signup, login, logout, and a session that persists across visits.

## Acceptance criteria
- [ ] A visitor creates an account with email and password
- [ ] A returning user logs in and stays logged in across visits
- [ ] A logged-in user can log out
- [ ] Passwords are stored hashed, never in plain text

## Non-goals
- No social login in v1
- No password reset flow yet
- No two-factor auth
```

Hand it to the agent with one request:

> Break specs/auth.md into atomic sub-issues. Respect the non-goals.

It proposes a breakdown. Prune it, then materialize the result to GitHub in one pass. The epic is created first, then each task with its parent and its blocker set at creation:

```
gh issue create --title "Epic: User authentication" --label epic --body-file specs/auth.md   # -> #1
gh issue create --title "User schema and password hashing"  --parent 1               --body-file .github/tasks/auth-02.md  # -> #2
gh issue create --title "Signup endpoint and form"          --parent 1 --blocked-by 2 --body-file .github/tasks/auth-03.md  # -> #3
gh issue create --title "Login and session persistence"     --parent 1 --blocked-by 2 --body-file .github/tasks/auth-04.md  # -> #4
gh issue create --title "Logout"                            --parent 1 --blocked-by 4 --body-file .github/tasks/auth-05.md  # -> #5
```

GitHub now holds the whole plan. The epic shows its children and their order:

```
#1  Epic: User authentication              [epic]
├─ #2  User schema and password hashing    ready
├─ #3  Signup endpoint and form            blocked by #2
├─ #4  Login and session persistence       blocked by #2
└─ #5  Logout                              blocked by #4
```

Only `#2` has no open blocker, so it is the single task ready to start. Close it, and `#3` and `#4` open up. Close `#4`, and `#5` follows. The spec stays in the repo as the record of intent. The issue graph runs the work, and the next task is always whatever has no open blocker.

That is the whole method. The principles below explain why each move is shaped the way it is.

## The Four Principles in Detail

### 1. Spec First

**Describe the outcome. Leave out the how.**

A spec records why an epic exists and what done looks like, and it stays out of the method. Agents reach for implementation detail early and bake decisions into a document meant to outlive them. A spec that records the outcome survives many implementations. A spec stuffed with steps rots on the first change of plan.

- Keep it to a page — Why, What, Acceptance criteria, Non-goals
- Write criteria as observable outcomes — the kind a test can check
- Leave the method to the implementer — the spec says what, not how
- Commit it to the repo — it travels with the code and diffs cleanly

**The test:** Could two engineers build from this spec and both satisfy it? If yes, it sits at the right altitude.

### 2. Graph Over Files

**GitHub's issue graph is the plan and the state.**

The breakdown lives as sub-issues under the epic. The order lives as blocking edges. Progress lives in open and closed. Nothing duplicates it in a file, so nothing drifts, and a context reset loses no plan because the plan was never in the context.

- Epic — a parent issue, marked with an `epic` label or an `Epic` issue type
- Tasks — sub-issues of the epic
- Order — blocking edges, between tasks and between epics alike
- Progress — read straight from the issues, visible to the whole team

**The test:** Can the state of the work be answered from the issues alone? If a file has to be checked, the graph is not yet carrying it.

### 3. Fence With Non-Goals

**The non-goals section does real work.**

Every line under Non-goals stops a sub-issue from forming and keeps an agent inside the scope of the epic. In the example, "no password reset yet" is the only reason a reset task did not appear in the breakdown.

- Name what the epic will not do, and where that work lives instead
- Treat deferred scope as future epics, never as quiet additions to this one
- Recheck the fence while pruning the derived sub-issues

**The test:** Does each non-goal prevent a plausible mistake? A fence around nothing is noise.

### 4. Derive, Then Work the Ready Set

**The model decomposes the spec. Work flows in unblocked order.**

Hand-writing every substep wastes effort and over-specifies the how. The agent proposes atomic sub-issues from the spec, a person prunes them, and the dependency graph decides what is ready. Each task carries its own throwaway plan, made fresh in the session and never written down.

- Derive sub-issues in plan mode, then review and cut before creating anything
- Materialize in one pass, with parent and blocking edges set at creation
- Start only issues with no open blockers, checking `blockedBy` first
- Record non-obvious choices in `DECISIONS.md`, the amendment log for when a spec shifts mid-flight

**The test:** Did the model write the substeps and a person approve them? If someone hand-wrote the task list, the spec was doing the agent's job.

## Why Lite

A lighter convention earns its place three ways. It removes a moving part, since no state file means nothing to sync and nothing to drift. It puts the work where people already look, so status is visible to teammates and other tools with no translation step. And it shrinks what an agent must learn: read the spec for the goal, query the graph for what is ready, write a throwaway plan for each task.

Heavier frameworks add phases, validators, and harnesses, and those pay off at large scale. For a solo developer or a small team, a spec plus the issue graph covers the same ground with far less to carry.

## Install

Drop the convention into a project:

```
curl -o CLAUDE.md https://raw.githubusercontent.com/<you>/spec-driven-github/main/CLAUDE.md
```

Append it to an existing `CLAUDE.md`:

```
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/<you>/spec-driven-github/main/CLAUDE.md >> CLAUDE.md
```

For Codex, Cursor, or Copilot, copy [AGENTS.md](AGENTS.md) alongside. It points at the same rules. Plugin packaging for `/plugin marketplace add` is on the roadmap.

## Requirements

A `gh` that ships the Issues 2.0 work (cli/cli#13057) for native `--parent` and `--blocked-by`. Earlier versions use the extension fallback described in CLAUDE.md.

Issue types need a GitHub organization. Without one, epics use a label, and everything else works the same.
