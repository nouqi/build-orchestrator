---
name: start
description: Start a build discussion session. Open-ended Q&A to pin down what to build, then suggests ending and writes the plan.
argument-hint: "[what you want to build]"
disable-model-invocation: true
---

The user wants to start a build discussion session for: $ARGUMENTS

Load the `build-orchestrator` skill (with the Skill tool) and follow it, on the **Broad** track whatever the request's size. Start the Phase 1 discussion loop right away. If the request above is empty, the first question is what they want to build.
