# build-orchestrator

A Claude Code skill that turns any build request (vague or precise) into a scoped plan, then runs it with tiered models: the main session plans with you, Sonnet triages and verifies, and Haiku does mechanical tasks in parallel. State lives in `.build/` (SPEC, MAP, PLAN) so agents never need to read the whole codebase.

## Install

```
git clone https://github.com/nouqi/build-orchestrator.git ~/.claude/skills/build-orchestrator
```

Update with `git pull` in that folder. Restart Claude Code after installing.

## Use

Ask Claude Code to build something, e.g. "build me a simple booking site". You can also invoke it directly with `/build-orchestrator`.
