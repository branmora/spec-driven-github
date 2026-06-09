# Example: one epic, end to end

This is the full version of the user-authentication epic from the README, with the task bodies and the working queries. The shape is the same for any epic: write the spec, derive the breakdown, materialize it, work the ready set.

## 1. The spec

`specs/auth.md`:

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

## 2. Derive

Ask the agent in plan mode:

> Break specs/auth.md into atomic sub-issues. Respect the non-goals.

Four tasks come back. The schema and hashing come first because signup and login both depend on it; logout depends on a session existing, so it follows login. Notice what the fence kept out: no reset task, no social-login task. Prune anything speculative before continuing.

## 3. Materialize

Create the epic, then the tasks in dependency order so every reference already exists. No organization here, so the epic uses a label:

```
gh issue create --title "Epic: User authentication" --label epic --body-file specs/auth.md   # -> #1
gh issue create --title "User schema and password hashing"  --parent 1               --body-file .github/tasks/auth-02.md  # -> #2
gh issue create --title "Signup endpoint and form"          --parent 1 --blocked-by 2 --body-file .github/tasks/auth-03.md  # -> #3
gh issue create --title "Login and session persistence"     --parent 1 --blocked-by 2 --body-file .github/tasks/auth-04.md  # -> #4
gh issue create --title "Logout"                            --parent 1 --blocked-by 4 --body-file .github/tasks/auth-05.md  # -> #5
```

A task body stays small and avoids the how. The acceptance criteria are the contract; the implementer decides the method at build time:

```
<!-- .github/tasks/auth-04.md -->
## Description
Authenticate a returning user and keep them logged in across visits.

## Acceptance criteria
- [ ] Valid email and password returns an authenticated session
- [ ] The session persists across browser restarts
- [ ] Invalid credentials are rejected without revealing which field was wrong

## Non-goals
- Does not handle signup (that is #3)
- Does not handle logout (that is #5)
```

## 4. The resulting graph

```
#1  Epic: User authentication              [epic]
├─ #2  User schema and password hashing    ready
├─ #3  Signup endpoint and form            blocked by #2
├─ #4  Login and session persistence       blocked by #2
└─ #5  Logout                              blocked by #4
```

## 5. Find ready work

Read the list of child issues, not the summary counter, which can freeze at the first sub-issue:

```
for n in $(gh issue view 1 --json subIssues -q '.subIssues[].number'); do
  blockers=$(gh issue view "$n" --json blockedBy -q '[.blockedBy[] | select(.state=="OPEN")] | length')
  state=$(gh issue view "$n" --json state -q .state)
  [ "$state" = "OPEN" ] && [ "$blockers" -eq 0 ] && echo "ready -> #$n"
done
```

Early on this prints `#2` alone. Close it and run again: `#3` and `#4` are ready. Close `#4`: `#5` is ready. The queue is derived from the graph, never maintained by hand.

## Cross-epic dependencies

Epics block other epics the same way tasks do. If a "User profile" epic cannot start until authentication ships:

```
gh issue edit <profile-epic#> --add-blocked-by 1
```

Containment and dependency are separate graphs. Parent and child say what is part of what; blocking edges say what waits on what, at any level.
