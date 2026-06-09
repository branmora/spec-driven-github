# Spec-Driven Development on GitHub

Run each epic from a short spec. GitHub's issue graph holds the breakdown, the order, and the state. No separate task file, no project management layer to maintain.

## The Four Principles

### 1. Spec First

**Describe the outcome. Leave out the how.**

A spec records why an epic exists and what done looks like. The method is decided at build time, by whoever implements it. A spec full of steps rots on the first change of plan; a spec that names the outcome survives many implementations.

- Keep it to a page — Why, What, Acceptance criteria, Non-goals
- Write criteria as observable outcomes — the kind a test can check
- Leave the method to the implementer — the spec says what, not how
- Commit it to the repo — it travels with the code and diffs cleanly

Turn vague intent into checkable criteria:

| Instead of... | Write... |
| --- | --- |
| "Add login" | "A returning user logs in and stays logged in across visits" |
| "Make it secure" | "Passwords are stored hashed, never in plain text" |

**The test:** Could two engineers build from this spec and both satisfy it? If yes, it sits at the right altitude.

### 2. Graph Over Files

**GitHub holds the plan and the state. No second tracker.**

The breakdown is sub-issues under the epic. The order is blocking edges. Progress is open and closed. Nothing duplicates it in a file, so nothing drifts, and a context reset loses no plan because the plan was never in the context.

- Epic — a parent issue, marked with an `epic` label or an `Epic` issue type
- Tasks — sub-issues of the epic, set with `--parent`
- Order — blocking edges set with `--blocked-by`, between tasks or between epics
- Progress — read from the issues, visible to the whole team without translation

**The test:** Can the current state of the work be answered from the issues alone? If a file has to be checked, the graph is not yet carrying it.

### 3. Fence With Non-Goals

**Name what this epic will not do.**

Each line under Non-goals stops a sub-issue from forming and keeps an agent inside scope. It is the cheapest scope control available, and it lives in the spec where the decomposition reads it.

- Say what is out of scope, and where that work lives instead
- Treat deferred scope as future epics, never as quiet additions to this one
- Recheck the fence while pruning the derived sub-issues

**The test:** Does each non-goal prevent a plausible mistake? A fence around nothing is noise.

### 4. Derive, Then Work the Ready Set

**The model breaks down the spec. The graph says what is next.**

Hand-writing every substep over-specifies the how. Let the agent propose atomic sub-issues, prune them, materialize them, then let the dependencies decide what is ready.

- Derive in plan mode — review and cut before creating anything
- Materialize in one pass — parent and blocking edges set at creation
- Start only unblocked issues — check `blockedBy` first, since GitHub serves a blocked issue without complaint
- Give each task its own throwaway plan, made in the session and never written down

**The test:** Did the model write the substeps and a person approve them? If someone hand-wrote the task list, the spec was doing the agent's job.

## The Loop

1. Write `specs/<epic>.md` using the template below.
2. Ask the agent to break it into atomic sub-issues that respect the non-goals.
3. Materialize the breakdown to GitHub in one pass.
4. Work the issues in unblocked order.
5. Record non-obvious choices in `DECISIONS.md`.

## Spec Template

```
# <Epic> — Spec
Status: Draft

## Why
The problem, in two to four sentences.

## What
What it does. The outcome, not the method.

## Acceptance criteria
- [ ] Testable, observable outcomes. These become the sub-issue seeds.

## Non-goals
The fence. Each line stops a sub-issue from forming.
```

## Materialize

Native flags, on a `gh` that ships cli/cli#13057. Create the epic, then the tasks in dependency order so every reference already exists:

```
gh issue create --title "Epic: <name>" --label epic --body-file specs/<epic>.md
gh issue create --title "<task>" --parent <epic#> --blocked-by <prev#> --body-file .github/tasks/<id>.md
```

Detect native support once:

```
gh issue create --help | grep -qE -- "--parent" && echo native || echo fallback
```

On an older `gh`, create the issues plainly, then link them with the gh-issue-ext extension or `gh api` GraphQL (`addSubIssue`, `addBlockedBy`):

```
gh issue-ext sub add <epic#> <task#>
gh issue-ext blocking add <blocked#> <blocker#>
```

## Blocking Discipline

- Place each edge where the constraint is real. When one task of epic B depends on epic A, block that single task.
- Skip redundant edges. An edge from epic A to epic B already covers B's children.
- Blocking is advisory. Read an issue's `blockedBy` before starting it.

## Finding Ready Work

Read the list of child issues, not the summary counter, which can freeze at the first sub-issue:

```
for n in $(gh issue view <epic#> --json subIssues -q '.subIssues[].number'); do
  blockers=$(gh issue view "$n" --json blockedBy -q '[.blockedBy[] | select(.state=="OPEN")] | length')
  state=$(gh issue view "$n" --json state -q .state)
  [ "$state" = "OPEN" ] && [ "$blockers" -eq 0 ] && echo "ready -> #$n"
done
```
