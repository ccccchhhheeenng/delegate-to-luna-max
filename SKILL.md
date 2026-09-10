---
name: delegate-to-luna-max
description: Orchestrate repository work with GPT-5.6 Sol at medium reasoning and delegate bounded implementation, test-writing, debugging, refactoring, documentation, or focused investigation tasks to GPT-5.6 Luna at max reasoning. Use proactively when a self-contained subtask has clear success criteria and delegation costs less than doing the full implementation locally; skip trivial edits, unclear work, broad architecture, and high-risk design decisions.
---

# Delegate to Luna Max

Use `gpt-5.6-sol` with `reasoning_effort=medium` as the orchestrator. The orchestrator understands the full request and repository architecture, decomposes work, prepares compact context, reviews and integrates changes, owns high-risk decisions, and performs final verification. This skill guides delegation; it cannot silently change the active parent model. If the active orchestrator is not Sol Medium and that distinction matters, disclose the mismatch instead of claiming this routing is active.

## Decide Whether to Delegate

Delegate only when the implementation or investigation is independent, bounded, and has observable success criteria, and when coordination costs less than completing it directly. Do not spawn a subagent merely to exercise this skill. Handle a change that takes only a few seconds directly.

Good Luna Max tasks include:

- Implementing one well-specified function, class, endpoint, or config option.
- Writing or extending unit and edge-case tests.
- Making small or medium changes across one or a few files using an established pattern.
- Performing repetitive refactors or mechanical transformations.
- Tracing an error, analyzing logs, or searching for a specific symbol or API usage.
- Fixing a common bug or updating focused documentation and comments.

Keep broad architecture redesigns, security-critical decisions, database migration strategy, ambiguous requirements, cross-subsystem debugging with high uncertainty, and work that depends on extensive implicit product context with Sol.

## Build a Compact Task Brief

Understand the full context first, then send only the minimum necessary context. Prefer summarizing context in the message over increasing `fork_turns`. Include:

```text
Goal:
<what to implement>

Relevant files:
<files or directories>

Context:
<minimum architecture context required>

Constraints:
<things that must not change>

Expected result:
<observable behavior>

Verification:
<tests, commands, or checks to run>

Completion:
<what the subagent must report after its work and verification are finished>
```

Do not include unrelated chat history. State repository conventions, ownership boundaries, and prohibited changes when relevant.

## Spawn Luna Max Correctly

Inspect the current `spawn_agent` tool schema and use its exact supported parameter names. For Luna delegation, explicitly set:

```text
fork_turns="none"
model="gpt-5.6-luna"
reasoning_effort="max"
```

Never omit `fork_turns` and never use `fork_turns="all"` for Luna delegation. Full-history forks inherit the parent model and reasoning effort and do not accept model or reasoning overrides.

Use a limited positive integer string such as `"1"`, `"2"`, or `"3"` only when inheriting that small amount of recent context is necessary. Compact the context into the task brief whenever possible.

Conceptual call shape, subject to the live tool schema:

```text
spawn_agent(
    task_name="<concise_task_name>",
    fork_turns="none",
    model="gpt-5.6-luna",
    reasoning_effort="max",
    message="<self-contained task brief>"
)
```

If the tool schema lacks a required override or the spawn fails, do not retry as an inherited or Sol subagent and do not silently fall back to a full-history fork. Tell the user Luna Max delegation did not occur, then let Sol take over or request direction when needed.

## Wait for Delegated Work

Every delegated task is joined work, not fire-and-forget background work. Record each spawned task name or agent identifier and wait for every agent required by the current request to reach a terminal state before reviewing, integrating, or sending the final response.

Use the live collaboration tool schema. When `wait_agent` is available, call it with a long bounded timeout and keep waiting until the required agent completes or needs attention. A timeout is only a progress checkpoint; it is not completion. If an agent is still running after a timeout, give the user a concise progress update when appropriate and wait again. Do not declare the parent task complete while required subagent output is pending.

If the subagent requests clarification or reports a recoverable problem, respond with `followup_task` or the appropriate messaging tool, then resume waiting. If the user cancels or replaces the work, explicitly interrupt the affected agent when the tool supports it rather than leaving it running.

Immediately before the final response, use `list_agents` or the available status tool to confirm that no agent spawned for the current request remains `running`. Never send the final response while a required child is active, and never allow its result to arrive after the parent has reported completion.

## Review and Integrate

Do not duplicate Luna's implementation while it is working. Sol may prepare non-overlapping review context while waiting, but review of Luna's result begins only after Luna reaches a terminal state. After it finishes, Sol must inspect the actual changes and verify at least:

- The result satisfies the original request and expected behavior.
- Changes follow repository conventions and contain no unnecessary edits.
- Other modules and contracts are not unintentionally affected.
- Syntax, types, tests, and relevant checks pass.
- There is no evident regression.

Run proportionate final verification yourself. Do not accept the subagent's summary as proof.

If Luna's result has a small error, lacks context, or followed an unclear instruction, send a focused correction with `followup_task` or create a new Luna Max task using the same explicit routing. If the failure reveals broad architecture work, cross-subsystem uncertainty, or a major design decision, have Sol take over instead of repeatedly asking Luna to guess.

## Context and Quota Discipline

Use this flow:

```text
User -> Sol Medium -> understand full context -> extract bounded task
     -> compact task brief -> Luna Max -> implement/test/investigate
     -> Sol Medium -> review/integrate/final verification
```

The compact handoff is part of the skill's purpose: it reduces unnecessary Sol implementation work without spending child context on full conversation history.
