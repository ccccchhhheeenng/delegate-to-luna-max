---
name: delegate-to-luna-max
description: Delegate ordinary bounded non-trivial repository implementation, tests, debugging, refactoring, documentation, or investigation to GPT-6 Luna at max reasoning. Use compact briefs, joined results, and evidence-based review. Keep unclear requirements, consequential decisions, tightly coupled, and high-risk work with the primary agent.
---

# Delegate to Luna Max

The primary agent scopes requirements, risk, relevant files, acceptance, and boundaries, then hands ordinary bounded non-trivial work to Luna before detailed diagnosis. Luna is the preferred executor and may choose a suitable local pattern or diagnose and fix within one boundary. The primary keeps unclear requirements, consequential decisions, integration, and acceptance. Route Luna explicitly at `max`.

## Select the workflow

Use this workflow automatically when its delegation criteria are met. Preserve `policy.allow_implicit_invocation: true`. An explicit user choice of another delegation workflow takes precedence for the same task; do not combine workflows. Merely mentioning, comparing, or editing a skill does not invoke its workflow.

## Spend less on coordination

Prefer one child for a bundled deliverable. At most three Luna children may be active concurrently for this task, counting every child role including verifiers. Choose one, two, or three only when each lane has independent useful work and disjoint files or resources; this is a ceiling, never a quota. Parallelism can increase total usage, so bundle related edits and targeted checks instead of splitting investigator, writer, and tester roles. Avoid speculative OPTIONAL lanes and routine review agents.

Keep briefs around 150-250 words and ordinary returns within 150 words when sufficient; never omit required boundaries or evidence to meet these targets. Send paths, relevant symbols, and changed facts rather than whole files, logs, skills, or repeated history. Batch independent scoped reads; reuse current evidence instead of rediscovering it. Store long logs in artifacts and return only outcomes, relevant errors, and paths.

## Delegate only useful, bounded work

Delegate ordinary bounded non-trivial work by default when its goal and acceptance can be stated compactly, including short, straightforward implementation, tests, documentation, and safe investigation. Skip only truly trivial work when the full coordination cost exceeds its benefit. Keep unclear requirements, consequential architecture decisions, high-risk or irreversible actions, and tightly coupled work with the primary agent; safe bounded evidence-gathering toward those decisions may still go to Luna. An explicit request to use Luna does not remove scope or safety boundaries.

Work in small successive waves within the three-child task cap and live available capacity. Read the tool's capacity semantics: a total-agent limit includes the primary agent; a child-only limit does not. Queue excess work. Never fill slots without independent useful work.

Before assigning writers, record existing changes using a scoped Git diff/status or file snapshots outside Git. Give parallel writers disjoint files and mutable resources; shared files, generated artifacts, services, and dependent tasks require sequencing. The primary agent may continue independent work, but must not duplicate a child's investigation, edits, or checks.

## Bound decisions before execution

For implementation, the primary agent supplies fixed requirements, acceptance criteria, safety boundaries, known relevant files, and any helpful local reference; it does not need to choose every implementation detail. Hand off before detailed diagnosis or solving. Luna may inspect the allowed files, use the first local pattern that fits, and combine diagnosis and repair within one explicit boundary. Keep unresolved user requirements and consequential architecture or risk decisions with the primary agent; Luna may gather bounded evidence and return it before crossing those decisions. Use a separate investigation lane only when its question and evidence are independently useful or a primary-owned decision must be made before implementation.

Apply these defaults unless the brief justifies a different limit:

- Follow the first existing pattern that meets acceptance. Do not compare alternatives, redesign abstractions, or reopen settled choices without contradictory evidence. If the chosen approach conflicts with correctness or safety, report the conflict instead of following it blindly.
- Investigations assess at most two evidence-backed hypotheses. Each further read must answer a named unresolved question within Allowed reads; stop discovery once enough evidence supports the assigned result. Report remaining uncertainty when the search limit is reached.
- After a failed assigned check, allow one focused repair by the same child and rerun the affected check once when the work remains safe and bounded; prefer this over primary takeover for convenience. If still failing, return the partial result and failure evidence; the primary agent chooses the next step under existing retry limits.
- Reopen a decision only for a new requirement, concrete contradictory evidence, or a failed check. Record hypothetical concerns briefly; they do not authorize more work. Material uncertainty blocking acceptance returns BLOCKED, never DONE.

These are observable workflow limits, not enforceable caps on internal reasoning tokens or elapsed time. High effort does not authorize additional scope. Once acceptance and assigned checks pass, return immediately.

## Give a compact brief

Read only enough authoritative context to set the boundary, then let Luna perform the scoped work. Use this brief, merging fields when that removes repetition:

```text
Goal: one deliverable and observable acceptance criteria
Context: exact relevant paths and existing/concurrent changes
Decisions: fixed requirements; allowed local choices; optional known reference; decisions reserved for the primary agent; or one bounded investigation question
Allowed reads: bounded paths, dependencies, commands, or sources
Allowed writes: exclusively owned files/resources, or none
Boundaries: prohibited actions; preserve others' changes; no nested delegation
Stop/budget: acceptance met and assigned checks complete; bounded search/check scope; report BLOCKED when exhausted
Checks: specific checks, one owner per check, and mandatory requirements
Dependencies: prerequisite results, or none; REQUIRED unless explicitly OPTIONAL
```

Choose a useful read boundary, including known local dependencies and applicable instructions, without authorizing a repository-wide scan. If additional access or an ownership conflict is needed, the child returns `BLOCKED` with the path/action and reason. The primary agent may amend the brief within the user's authorized scope; ask the user only for genuinely missing authority or requirements.

Children must preserve pre-existing and concurrent changes, never revert others' work, and never spawn agents unless the user explicitly authorizes nested delegation. Stop after the deliverable and assigned checks; no opportunistic cleanup or open-ended verification.

Require a concise, evidence-backed return:

```text
Status: DONE | DONE_WITH_CONCERNS | BLOCKED | FAILED
Files: changed paths or none
Result: behavior or findings
Checks: command, outcome, and relevant artifact/output; or none
Concern: unresolved issue, assumption, missing scope, or none
```

`DONE_WITH_CONCERNS` means the deliverable and assigned checks are complete with a specific residual concern. Missing mandatory checks or incomplete acceptance cannot be labeled `DONE`; return `BLOCKED` or `FAILED` with the partial result. A completed investigation may report an unresolved finding.

## Route explicitly

Inspect the live collaboration schema and set:

```text
fork_turns="none"
model="gpt-6-luna"
reasoning_effort="max"
```

Use a small positive history fork only when essential recent context cannot be expressed in the brief and the tool supports explicit overrides with it. Never use a full-history fork or omit the model/effort. If routing is unavailable, disclose it and do the work in the primary agent; do not substitute an inherited-model child.

On capacity or transient rate-limit failure, wait for capacity or the indicated retry window before one retry with the same routing. For unsupported parameters/model, do not retry blindly. Track the actual child id, ownership, priority, and status. Tool acceptance confirms the requested routing; when actual model execution must be proven, use available runtime metadata, not a child's self-description.

## Join and recover

Every child is joined work. Review a completed lane and release its dependent work as soon as its prerequisites are terminal and accepted, provided remaining active lanes have disjoint files/resources. Do not impose an all-wave barrier on unrelated work. A progress message or timeout is not completion. Prefer event-driven waits over repeated polling; use bounded waits compatible with user updates. Inspect status only when completion or ownership is unclear. Silence alone does not justify interruption.

If progress is unclear after checking available status/activity, send one non-interrupting message requesting completed work, current work, and blockers, or correcting scope drift. Allow a response at a message boundary. Do not repeat pings or interrupt solely for slowness.

Use `send_message` to steer a running child. For an idle/terminal child needing another turn, use `followup_task` or the live equivalent that starts a turn; a message alone may not restart it. Keep follow-ups within an explicit brief. For a recoverable block, revise the brief and retry only the unfinished work once, preferably with the same child when safe and bounded and within existing retry limits. If the same cause recurs, the primary agent takes over.

When a child terminates without a report, inspect existing artifacts and check evidence first. Recover the missing report/check once only if needed; do not rerun completed implementation. Before taking over owned files, confirm the previous writer has stopped.

Cancel affected children if the user cancels or replaces the task. Before finalizing, collect every required result and explicitly stop any unneeded OPTIONAL work. Confirm none of this request's children remain active; a requested interruption is not itself proof they stopped.

## Accept once, using evidence

Inspect the actual diff/artifact against the recorded starting state, acceptance criteria, ownership, and preserved user changes. Review assigned check evidence and material concerns, not just the child's summary. Use one acceptance pass by default: one artifact/diff review plus mandatory checks and the smallest check needed to establish changed behavior. Assign each check one owner. Do not reproduce the child's diagnosis or redo correct work and passing checks. Skip optional suites and independent verifiers unless a concrete unresolved correctness concern justifies them; any verifier counts toward the three-child cap.

Do not repeat passing checks unless later edits invalidate them, evidence is missing, or a specific unresolved risk warrants it. Required user/repository checks still apply. Repair correctness and scope issues before claiming completion; disclose unavailable checks. Keep major design uncertainty with the primary agent.

Briefly report the result, changed files, checks and limitations, and actual child count/effort. Mention retries or capacity fallback only if they occurred. Do not claim measured speed or token savings without evidence.
