# Delegation Contract

Read this file before writing any worker brief and before reviewing worker handoffs. It standardizes the GPT-to-worker contract and the evidence workers must return.

## Rules

- GPT turns each request into an implementation-ready brief. Never send vague goals, unresolved boundaries, or implicit acceptance criteria.
- Delegate only substantial, clearly bounded scopes with non-overlapping ownership.
- Choose every spawn target from `route-catalog.md`; use only runtime-registered agent types.
- Parallelize only independent scopes; sequence dependent scopes and hand downstream workers the stabilized contract.
- Workers adapt to concurrent edits and never revert or overwrite changes made by other agents.
- No worker declares release, acceptance, production readiness, or numerical certification; only GPT approves after unified review.

## Worker brief template

Copy the following block and fill every field. If a field cannot be resolved, either resolve it as GPT or record the assumption explicitly in the brief before delegating.

```text
TASK_NAME: <spawn task name>
AGENT_TYPE: <runtime-registered agent type from route-catalog.md>

OBJECTIVE:
- What must this worker accomplish, in one or two sentences.

CONTEXT:
- Relevant repository layout, prior decisions, and artifacts the worker needs.

CURRENT BEHAVIOR:
- What exists today and why it is insufficient.

DESIRED BEHAVIOR:
- Observable end state after this scope is complete.

OWNED FILES/MODULES:
- Exact paths the worker may modify. Everything else is off-limits.

OWNERSHIP BOUNDARIES:
- Which interfaces, algorithms, and decisions belong to GPT or Luna and must not be changed here.

INPUTS:
- Inputs, fixtures, environment variables, or commands the worker may use.

EXPECTED OUTPUTS:
- Files, artifacts, output formats, and exit contracts.

INVARIANTS AND INTERFACE CONTRACTS:
- Numerical invariants, precision thresholds, state/rollback semantics, certificate meaning, and API compatibility that must be preserved.

NUMERICAL REQUIREMENTS (if applicable):
- Solver, KKT, precision, residual, or certificate checks in scope, with exact thresholds.

FORBIDDEN CHANGES:
- Core algorithms, architecture, contracts, and unrelated files that must not change.

DEPENDENCIES:
- Ordering relative to other workers and the stabilized contract to consume.

EDGE CASES:
- Known failure modes, boundary inputs, and concurrency conditions to handle.

ACCEPTANCE CHECKS:
- Exact commands and expected results that define done for this scope.

REQUIRED EVIDENCE:
- Tests, static checks, cluster job IDs, residuals, certificates, or repro commands to return.

ESCALATION CONDITIONS:
- When to stop and return to GPT instead of proceeding.

EXPECTED HANDOFF:
- Use the worker handoff template below, unchanged structure.
```

## Worker handoff template

Require every worker to return this structure:

```text
## Summary
<what changed and why, 2-5 sentences>

## Changed files/modules
- path: change and reason

## Invariants and contracts preserved
- list each invariant or contract and how it remains valid

## Verification evidence
- command 1: result
- command 2: result
- numerical residuals/certificate checks/cluster job IDs when applicable

## Risks, assumptions, skipped checks
- list

## Decisions requested from GPT
- list or "none"
```

## GPT acceptance evidence template

Use this when reviewing the combined result; record one verdict per row and keep the filled checklist as the acceptance record.

```text
Scope: <brief id or task name>

| Check | Evidence | Verdict |
| --- | --- | --- |
| Implementation matches declared architecture and boundaries | ... | pass/fail |
| Core and peripheral integration without ownership or contract conflicts | ... | pass/fail |
| Unit, regression, static, and end-to-end checks | ... | pass/fail |
| PBS/HPC runs complete when in scope | ... | pass/fail |
| Precision thresholds and original-coordinate certificates/residuals | ... | pass/fail |
| Rollback, failure paths, state transitions, certificate semantics | ... | pass/fail |
| Material risks and skipped checks accounted for | ... | pass/fail |

FINAL DECISION: approve / follow-up required / reject
```

## Escalation paths

- DSflash finds a likely core defect or required contract change: stop that portion, return the evidence and proposed escalation; GPT decides.
- Luna needs a design deviation outside the supplied architecture: report to GPT before proceeding.
- A worker cannot complete within its ownership: stop, report the blocker, and ask GPT for a decision; do not expand scope.
