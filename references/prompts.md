# Formats and prompt templates

Fill in the `<...>` parts. Keep every prompt self-contained: subagents don't see the conversation.

## PLAN.md format

```markdown
# Plan: <project>
Spec: .build/SPEC.md · Map: .build/MAP.md

## Wave 1
### T1: <short title>
- tier: junior | senior        (filled by triage)
- files: <paths to create/change>
- reads: <extra paths to read, if any>
- goal: <one or two sentences, outcome not steps>
- accept: <command or observable check>
- status: todo | in_progress | review | done | blocked

## Wave 2
...
```

## Map builder (Junior)

```
Read the repository at <root> and write .build/MAP.md, under 150 lines, so that other agents
can work on single tasks without exploring. Include:
1. Stack, plus the install/run/test/build commands (verify they exist in package files or config).
2. Directory tree of key paths only (skip node_modules, build output, assets).
3. One line per important module: path — what it does.
4. Shared interfaces: key types, API routes, DB tables, env vars.
5. Conventions you can see: naming, error handling, styling, test layout.
Don't paste code. Reply only with "MAP written" plus any part of the repo you couldn't understand.
```

## Triage (Senior)

```
You are the senior engineer triaging a build plan. Read .build/PLAN.md and .build/MAP.md
(SPEC.md only if a task is unclear). For every task, decide the tier:
- junior: pattern exists or is fully specified, mechanical logic, mistakes would be obvious and contained.
- senior: design decisions, cross-cutting changes, security/auth/payments, concurrency,
  tricky logic, unknown-cause debugging, or subtle-failure risk.
Also: split any task that needs more than ~3 changed files or has no concrete acceptance
check, and flag any two tasks in the same wave that touch the same file.
Edit PLAN.md in place (set `tier:`, apply splits). Reply with at most 15 lines:
one line per task, "T<n>: <tier> — <reason in a few words>", then any splits or conflicts.
```

## Worker (Junior or Senior)

```
Task <id> from .build/PLAN.md. Project root: <root>.
Read .build/MAP.md first, then only the files listed for this task (plus anything they
directly import, if needed). Don't explore beyond that. If you must, say why in your report.

Goal: <goal>
Files: <files>
Acceptance: <accept>
Constraints: follow the conventions in MAP; no new dependencies unless the task says so;
don't change files outside the list without reporting it.

Do the task, then run the acceptance check. Reply in this format only:
STATUS: DONE | BLOCKED | NEEDS_DECISION
CHANGED: <paths>
CHECK: <command run and result, one line>
INTERFACE CHANGES: <new or changed exports/routes/types/env vars, or "none">
NOTES: <at most 3 lines: risks, assumptions, the question if NEEDS_DECISION>
```

## Verifier (Senior)

```
Verify task(s) <ids> from .build/PLAN.md. Project root: <root>.
For each task: read its goal and acceptance check, read only the changed files
(<paths per task>), and run the acceptance check yourself; don't trust the worker's claim.
Check: does it meet the goal, does it follow MAP conventions, any obvious bug, security
issue or unhandled error, anything changed outside its file list.
Fix trivial issues (typos, a missing import) directly and say so. Don't fix real defects;
report them.
Reply per task, at most 5 lines each:
T<n>: PASS | FAIL
- <finding with file:line, if FAIL or fixed>
```
