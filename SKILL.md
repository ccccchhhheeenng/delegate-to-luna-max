---
name: delegate-to-luna-max
description: Use for non-trivial repository work with independent, bounded implementation, tests, debugging, refactoring, documentation, or focused investigation. The primary agent delegates suitable work to GPT-6 Luna at max reasoning with compact context, waits for required results, then reviews. Skip trivial, ambiguous, tightly coupled, architecture-wide, and high-risk work.
---

# Delegate to Luna Max

The current primary agent is the higher-capability orchestrator for this workflow. It owns decomposition, high-risk decisions, integration, and final verification. Apply the workflow without checking or claiming a specific primary model or reasoning effort. This skill does not change the active primary model; explicitly route only the delegated work to Luna Max.

## Explicit Workflow Precedence

If the user explicitly invokes `$delegate-to-luna-adaptive`, do not apply this workflow to the same task even when project or global instructions caused this skill to load. The adaptive workflow owns that turn.

If the user explicitly invokes `$luna-task-owner`, do not apply this workflow to the same task even when project or global instructions caused this skill to load. The explicit skill owns that turn. Return to this workflow only when the user explicitly selects `$delegate-to-luna-max`, or after the task-owner workflow reports that the task is ineligible and the user chooses the standard workflow.

## Discovery and Always-On Use

Implicit skill invocation is heuristic: a clear description improves selection but does not guarantee this file loads for every task. Keep `policy.allow_implicit_invocation: true` in `agents/openai.yaml`. When the user wants this workflow considered in every new Codex run, add a short global `$CODEX_HOME/AGENTS.md` instruction that tells Codex to read this `SKILL.md` before non-trivial repository work. Keep the full workflow here rather than duplicating it in `AGENTS.md`.

## Decide Whether to Delegate

Delegate only independent, bounded work with observable success criteria when coordination costs less than doing it directly. Handle trivial, ambiguous, tightly coupled, architecture-wide, security-critical, or high-risk work in the primary agent. Keep the workflow one level deep; Luna must not spawn another subagent unless the user explicitly requests nested delegation.

Plan small successive waves. Each child gets one primary deliverable, an explicit scope, and a required or optional result label. A wave may contain up to three Luna Max agents, but use fewer when there are fewer truly independent tasks, when sequencing is safer, or when fewer child slots are available. Never spawn three merely to fill capacity. Compute the wave size from the independent task count and available child slots (the configured child-slot limit excludes the primary agent), capped at three. If capacity is below three, degrade to the available capacity and explicitly report that limitation; never claim that three agents ran.

Parallel writing is allowed only with explicitly disjoint file ownership in every brief. Tasks that depend on one another, touch the same file, or require a shared mutable artifact run sequentially. Do not ask a child to gather unrelated repository context or perform a broad scan. Suitable tasks include a well-specified function/config option, focused tests, a mechanical refactor, a targeted symbol/API investigation, or focused documentation.

## Build a Compact Task Brief

The primary agent reads enough authoritative context to define the boundaries, then sends only the minimum context needed. Use exact paths rather than a repository-wide directory when possible. Every brief must contain all of these fields:

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

Stop when:
<the expected result is reached and the specified verification is complete; report immediately rather than extending the task>

Verification:
<specific tests, commands, or checks to run; state any justified conditions for additional checks>

Result priority:
REQUIRED | OPTIONAL
```

Do not include unrelated chat history. Once the stop condition is met, the child reports without adding refactors, tests, or investigation. After specified checks pass, broaden verification only for a failed check or a concrete new risk within Allowed reads; explain why in the return. A child must return only this fixed contract after its work and checks (use `none` when a field is empty):

```text
Status: DONE | DONE_WITH_CONCERNS | BLOCKED | FAILED
Changed files: <paths or none>
Behavior implemented or Findings: <summary>
Tests/checks run: <commands and outcomes>
Assumptions: <assumptions or none>
Unresolved issues: <issues or none>
Evidence: <paths, commands, outputs, or hashes>
```

Use `DONE_WITH_CONCERNS` when the deliverable and verification are complete but a specific doubt remains. State that doubt under Unresolved issues and let the primary agent decide whether more work is needed; do not investigate it indefinitely. Use `BLOCKED` when the deliverable cannot be completed within the brief.

## Spawn Luna Max Correctly

Inspect the live `spawn_agent` schema and use its exact parameter names. For every Luna delegation, explicitly set:

```text
fork_turns="none"
model="gpt-6-luna"
reasoning_effort="max"
```

Never omit `fork_turns` and never use `fork_turns="all"`. Use `fork_turns="none"` for Luna by default. A limited positive integer string such as `"1"`, `"2"`, or `"3"` is allowed only when a specific small amount of recent context is genuinely necessary and cannot be expressed cleanly in the compact brief. A parallel writer must receive a non-overlapping ownership list. If a required override is unavailable or spawning fails, do not retry with an inherited-model subagent or a full-history fork; disclose that Luna Max delegation did not occur and let the primary agent take over.

## Wait for Waves

Every spawned child is joined work, not fire-and-forget. Record its task identifier and whether the result is REQUIRED or OPTIONAL. The primary agent must not implement overlapping work while any child in that scope is running. Wait for every REQUIRED result to reach a terminal state before reviewing, integrating, starting a dependent wave, or sending the final response. OPTIONAL work never silently becomes required; if it is not needed, cancel or drop it explicitly and ensure no child remains running before finalizing.

Use the live collaboration status/wait tools. A timeout is only a progress checkpoint: inspect status and continue waiting while a required child is active. If a child is taking much longer than its bounded task suggests, check its live status and available thread activity or work artifacts before drawing conclusions. Silence alone does not mean it is stuck. If progress is still unclear, send one short, non-interrupting message asking what it has completed, what it is doing now, and whether it is blocked; ask it to continue within the original scope. Give it time to reach a message boundary and reply. Avoid repeated pings or overlapping investigation. Do not interrupt a child merely because it is slow.

If the child appears to be expanding into low-value work, send a focused, non-interrupting course correction that restates the deliverable, scope, and stopping condition. Do not request a new task or broaden its permissions. If a child requests clarification, reports a recoverable issue, or reaches BLOCKED, send a focused follow-up only within the original scope; otherwise let the primary agent take over. If the user cancels or replaces the work, interrupt affected children when supported.

## Review and Integrate

After required children are terminal, the primary agent inspects actual changes and verifies the original behavior, file ownership, absence of unrelated edits, syntax/types/tests, and relevant regressions. Do not accept a child summary as proof. For `DONE_WITH_CONCERNS`, assess the stated doubt and resolve any concern that affects correctness or scope before integration. Correct small errors with a focused sequential follow-up using the same Luna routing; keep broad uncertainty or major design decisions with the primary agent. Immediately before finalizing, confirm no child spawned for this request remains running and report actual wave sizes plus any capacity fallback.

## Operating Shape

```text
User -> primary agent -> bounded task briefs
     -> successive waves of 1-3 independent Luna Max children
     -> wait for required results -> primary agent review/integration/final verification
```

The compact handoff prevents irrelevant discovery and overlapping primary-agent implementation while preserving Luna Max's focused execution.
