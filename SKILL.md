---
name: delegate-to-luna-max
description: Use for any non-trivial repository task containing bounded implementation, test-writing, debugging, refactoring, documentation, or focused investigation. Orchestrate with GPT-5.6 Sol at medium reasoning; delegate suitable work to GPT-5.6 Luna at max reasoning with compact context, wait for every required result, then review. Skip trivial, ambiguous, tightly coupled, architecture-wide, and high-risk work.
---

# Delegate to Luna Max

Use `gpt-5.6-sol` with `reasoning_effort=medium` as the orchestrator. Sol owns decomposition, high-risk decisions, integration, and final verification. This skill guides delegation; it cannot silently change the active parent model. If the active orchestrator is not Sol Medium and that distinction matters, disclose the mismatch instead of claiming this routing is active.

## Discovery and Always-On Use

Implicit skill invocation is heuristic: a clear description improves selection but does not guarantee this file loads for every task. Keep `policy.allow_implicit_invocation: true` in `agents/openai.yaml`. When the user wants this workflow considered in every new Codex run, add a short global `$CODEX_HOME/AGENTS.md` instruction that tells Codex to read this `SKILL.md` before non-trivial repository work. Keep the full workflow here rather than duplicating it in `AGENTS.md`.

## Decide Whether to Delegate

Delegate only independent, bounded work with observable success criteria when coordination costs less than doing it directly. Handle trivial, ambiguous, tightly coupled, architecture-wide, security-critical, or high-risk work in Sol. Keep the workflow one level deep; Luna must not spawn another subagent unless the user explicitly requests nested delegation.

Plan small successive waves. Each child gets one primary deliverable, an explicit scope, and a required or optional result label. A wave may contain up to three Luna Max agents, but use fewer when there are fewer truly independent tasks, when sequencing is safer, or when fewer child slots are available. Never spawn three merely to fill capacity. Compute the wave size from the independent task count and available child slots (the configured child-slot limit excludes Sol), capped at three. If capacity is below three, degrade to the available capacity and explicitly report that limitation; never claim that three agents ran.

Parallel writing is allowed only with explicitly disjoint file ownership in every brief. Tasks that depend on one another, touch the same file, or require a shared mutable artifact run sequentially. Do not ask a child to gather unrelated repository context or perform a broad scan. Suitable tasks include a well-specified function/config option, focused tests, a mechanical refactor, a targeted symbol/API investigation, or focused documentation.

## Build a Compact Task Brief

Sol reads enough authoritative context to define the boundaries, then sends only the minimum context needed. Use exact paths rather than a repository-wide directory when possible. Every brief must contain all of these fields:

```text
Goal:
<one primary deliverable>

Relevant files:
<exact files or narrowly scoped directories>

Context:
<minimum architecture context required>

Allowed reads:
<exact paths, commands, or data sources>

Allowed writes:
<exact files this child owns, or "none" for read-only work>

Do not:
<files, systems, actions, or scope that are prohibited>

Scope expansion gate:
If the work needs anything outside Allowed reads or Allowed writes, stop and return BLOCKED with the missing scope and reason. Do not explore, infer permission, or modify beyond scope.

File ownership:
<exclusive write ownership; state "read-only" when applicable>

Expected result:
<observable behavior or findings>

Verification:
<tests, commands, or checks to run>

Result priority:
REQUIRED | OPTIONAL
```

Do not include unrelated chat history. A child must return only this fixed contract after its work and checks (use `none` when a field is empty):

```text
Status: DONE | BLOCKED | FAILED
Changed files: <paths or none>
Behavior implemented or Findings: <summary>
Tests/checks run: <commands and outcomes>
Assumptions: <assumptions or none>
Unresolved issues: <issues or none>
Evidence: <paths, commands, outputs, or hashes>
```

## Spawn Luna Max Correctly

Inspect the live `spawn_agent` schema and use its exact parameter names. For every Luna delegation, explicitly set:

```text
fork_turns="none"
model="gpt-5.6-luna"
reasoning_effort="max"
```

Never omit `fork_turns` and never use `fork_turns="all"`. Use `fork_turns="none"` for Luna by default. A limited positive integer string such as `"1"`, `"2"`, or `"3"` is allowed only when a specific small amount of recent context is genuinely necessary and cannot be expressed cleanly in the compact brief. A parallel writer must receive a non-overlapping ownership list. If a required override is unavailable or spawning fails, do not retry with an inherited or Sol subagent or a full-history fork; disclose that Luna Max delegation did not occur and let Sol take over.

## Wait for Waves

Every spawned child is joined work, not fire-and-forget. Record its task identifier and whether the result is REQUIRED or OPTIONAL. Sol must not implement overlapping work while any child in that scope is running. Wait for every REQUIRED result to reach a terminal state before reviewing, integrating, starting a dependent wave, or sending the final response. OPTIONAL work never silently becomes required; if it is not needed, cancel or drop it explicitly and ensure no child remains running before finalizing.

Use the live collaboration status/wait tools. A timeout is only a progress checkpoint: inspect status and continue waiting while a required child is active. If a child requests clarification, reports a recoverable issue, or reaches BLOCKED, send a focused follow-up only within the original scope; otherwise let Sol take over. If the user cancels or replaces the work, interrupt affected children when supported.

## Review and Integrate

After required children are terminal, Sol inspects actual changes and verifies the original behavior, file ownership, absence of unrelated edits, syntax/types/tests, and relevant regressions. Do not accept a child summary as proof. Correct small errors with a focused sequential follow-up using the same Luna routing; keep broad uncertainty or major design decisions with Sol. Immediately before finalizing, confirm no child spawned for this request remains running and report actual wave sizes plus any capacity fallback.

## Operating Shape

```text
User -> Sol Medium -> bounded task briefs
     -> successive waves of 1-3 independent Luna Max children
     -> wait for required results -> Sol review/integration/final verification
```

The compact handoff prevents irrelevant discovery and overlapping Sol implementation while preserving Luna Max's focused execution.
