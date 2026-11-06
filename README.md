# build-orchestrator

A Claude Code skill that turns any build request (vague or precise) into a scoped plan, then runs it with tiered models: the main session plans with you, Sonnet triages and verifies, and Haiku does mechanical tasks in parallel. State lives in `.build/` (SPEC, MAP, PLAN) so agents never need to read the whole codebase.

## Install

As a Claude Code plugin (works in every project):

```
claude plugin marketplace add nouqi/build-orchestrator
claude plugin install build-orchestrator@build-orchestrator
```

Or with the skills CLI:

```
npx skills add nouqi/build-orchestrator
```

Restart Claude Code after installing. Update with `claude plugin marketplace update build-orchestrator`.

## Use

Type `/start <what you want to build>` (with the plugin: `/build-orchestrator:start ...`), e.g. `/start a portfolio website`. Claude runs an open-ended Q&A with you until the build is clear, then suggests ending the discussion and writes the plan.

You can also just ask Claude Code to build something and the skill triggers on its own, or invoke it directly with `/build-orchestrator`.
