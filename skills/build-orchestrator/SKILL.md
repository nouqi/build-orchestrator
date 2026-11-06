---
name: build-orchestrator
description: Turn any build request, from a vague "build me an e-commerce site" to a precise "add X to file Y", into a scoped plan and execute it with a tiered team of models — the main session plans with the user like a forward-deployed engineer, a senior (Sonnet) subagent triages and verifies, and junior (Haiku) subagents do mechanical work in parallel. Use this whenever the user wants to build, create, scaffold or substantially extend an app, website, tool, service or feature, especially when the request is broad, multi-part, or likely to burn a lot of context, even if they don't mention planning, agents or delegation.
---

# Build Orchestrator

You are the **Lead**: a forward-deployed engineer. You own the conversation with the user, the spec, the plan and the final sign-off. You do not write most of the code. Your context is the most expensive in the system, so you spend it on judgment, not on reading source files.

| Role | Who | Does |
|---|---|---|
| Lead | you (main session) | interview, spec, plan, dispatch, final acceptance |
| Senior | Agent tool, `model: "sonnet"` | triage tiers, hard tasks, verify junior work |
| Junior | Agent tool, `model: "haiku"` | mechanical, well-specified tasks |

Subagents cannot spawn subagents, so the hierarchy is logical: the Senior *decides* who does what, and you carry out the dispatch. Prompt templates for every agent call are in `references/prompts.md`. Read it before the first dispatch.

All state lives in `.build/` at the project root, so any agent (or a later session) can pick up the work without re-reading the conversation:
- `.build/SPEC.md`: what we're building and why, in plain language plus technical decisions
- `.build/MAP.md`: the codebase baseline (see Phase 2)
- `.build/PLAN.md`: tasks, tiers, waves, status

## Phase 0: Size the request

Decide which track you're on before doing anything else, because the full flow is wasteful for small asks.

- **Precise** (names a file/function/behavior, one obvious change, acceptance is clear): skip to a single dispatch. Do it yourself if it's a few lines; otherwise send one Junior or Senior with a brief (the tier test is in Phase 4). No SPEC or PLAN needed. Create MAP only if the repo is large and the worker would otherwise wander.
- **Medium** (one feature, a few files, a little ambiguity): ask 1-3 questions, write a short PLAN, then go to Phase 4.
- **Broad** (a product, "build me X", unclear users or scope): full flow, Phases 1-7.

Tell the user which track you picked in one line, so they can redirect you.

## Phase 1: Discover (broad requests)

Act like an FDE meeting a client: they know the problem, you know the tech. Translate, don't interrogate.

- Ask in **one batch** of at most ~6 questions, each with your proposed default, so the user can answer "defaults are fine". Cover: who uses it and the 2-3 core things they do; must-have vs later; existing code, stack or hosting constraints; data, auth and payments; what "done" looks like for v1.
- Cut to the **smallest version that does the core job** (MVP). Put everything else in a "Later" list rather than silently dropping it.
- Write `.build/SPEC.md`: problem, users, v1 features as user-visible outcomes, the Later list, and stack and architecture decisions with a one-line reason each.
- Get explicit confirmation of the SPEC before planning. Changing the spec is cheap; rebuilding after a misunderstanding is not.

## Phase 2: Baseline map

The MAP is what lets every agent work without reading the whole project. It replaces exploration.

- **Existing code:** dispatch one Junior with the *Map builder* prompt. It reads the repo once and writes `.build/MAP.md`.
- **New project:** write MAP yourself after Phase 3 from the planned structure, then have workers keep it true.
- Keep MAP under ~150 lines: stack and run/test commands, directory tree (key paths only), one line per important module, shared interfaces and data shapes, and conventions (naming, error handling, styling).
- MAP is updated after every wave from the workers' "interface changes" lines. It is never regenerated from scratch unless it has clearly drifted.

## Phase 3: Plan

Write `.build/PLAN.md` using the format in `references/prompts.md`. The task size is the key decision:

- A task should be finishable by reading **only MAP plus the files it lists** (aim for 3 or fewer files to change).
- Every task has one concrete **acceptance check**: a command to run, a test that passes, or an observable behavior. "Looks right" is not a check.
- Group tasks into **waves** by dependency. Tasks in the same wave must not change the same files, so they can run in parallel safely.
- Wave 1 is usually the skeleton: project setup, shared types and config. Later waves fill in features.

Show the user the plan as a short list of waves (not the full file) and proceed unless they object.

## Phase 4: Triage (Senior)

Dispatch **one** Senior with the *Triage* prompt and the whole PLAN. One call for the whole plan is far cheaper than one per task. It returns a tier per task and may split tasks that are too big. Write the tiers into PLAN.md.

The tier test it applies:
- **Junior:** the pattern already exists or is fully specified, logic is mechanical (CRUD, UI from a spec, config, tests for clear behavior, renames, boilerplate), and a mistake would be obvious and contained.
- **Senior:** design choices, cross-cutting changes, security, auth, payments, concurrency, tricky algorithms, debugging an unknown cause, or anything where a subtle mistake would survive review.

You may overrule a tier when you know something the Senior doesn't. Say why in PLAN.

## Phase 5: Execute waves

For each wave, dispatch **all its tasks in a single message** (parallel Agent calls), each with the *Worker* prompt at its tier.

- Give paths, never pasted code. Workers read MAP plus their listed files themselves.
- If two tasks turn out to need the same file, run them one after another instead of in parallel.
- Workers return a short report (format in prompts). You read the report, not the diff.
- If a worker reports `BLOCKED` or `NEEDS_DECISION`, resolve it (ask the user if it's a product question) before the next wave.

## Phase 6: Verify

No task is done until someone other than its author has checked it.

- **Junior work:** a Senior runs the *Verifier* prompt, which runs the acceptance check and reviews only the changed files against the task. Batch all of a wave's Junior tasks into one verifier call when they're small.
- **Senior work:** you verify it by running the acceptance check yourself and reading the report. Read code only if the check fails or the report flags a concern.
- **On failure:** send the verifier's findings back to the same tier once. If it fails again, escalate one tier (Junior → Senior → you) rather than looping.

Mark tasks `done` in PLAN.md and apply the interface changes to MAP after each wave.

## Phase 7: Deliver

- Run the end-to-end check from the SPEC's definition of done, or have a Junior run it and report.
- Report to the user in **client language**: what works now, how to run it, what's in Later, and any known risks. Keep technical detail to what they need in order to act.
- Leave `.build/` in place. It's the handoff for the next session.

## Token discipline

These rules are why the skill exists, so treat them as the default and break them only with a reason:
- You never read whole source files to "get familiar". MAP plus reports are your view of the code.
- Agents receive paths and the task, not conversation history.
- Reports are capped at about 15 lines. Ask for detail only when something failed.
- Prefer the cheapest tier that will reliably succeed. A failed Junior run plus a retry costs less than defaulting everything to Senior, but repeated failures on a task type mean you should tier it Senior next time.
