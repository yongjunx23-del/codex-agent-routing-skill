---
name: codex-agent-routing-skill
description: Use this skill to decompose non-trivial tasks, delegate bounded scopes across GPT, Luna, and DSflash workers, and choose runtime-registered official, direct-API, subscription/bridge, or user-custom provider router agent types. Trigger on task decomposition, multi-agent delegation, GPT/Luna/DSflash division of labor, API-backed or router agent selection, custom provider or API routing, parallel worker execution, result aggregation, model or route retention decisions, or final release decisions requiring tests, cluster runs, and numerical certificate or residual evidence.
---

# Codex Agent Routing Skill

## Purpose

Route non-trivial implementation, debugging, numerical, HPC, review, and refactoring work across runtime-registered agent types, including official/native routes, direct provider API routes, and subscription/bridge router routes. GPT remains accountable for the whole task and is the only final reviewer. Do not invoke this skill for trivial questions, tiny edits, or work whose coordination cost exceeds its implementation cost.

## Roles and ownership

### GPT: architect and sole final reviewer

- Define the architecture, numerical invariants, task boundaries, risk classification, and acceptance criteria before delegating.
- Decompose work into owned scopes, classify candidate routes, and decide parallel versus sequential execution.
- Resolve underspecification before delegation; ask the user only when a missing decision materially changes the result, otherwise record the assumption in the brief.
- Review every worker result against tests, relevant cluster runs, and original-coordinate certificates or residuals when numerical solvers are involved.
- Aggregate outputs from all routes, cross-review conflicting evidence, and make the only final release, acceptance, and production-readiness decision.

### Luna: core engineer

- Own high-risk core changes: solver paths, KKT systems, numerical precision, convergence and rollback behavior, mutable solver state, recovery logic, and certificate semantics.
- Preserve stated numerical invariants and compatibility contracts.
- Change core algorithms only inside the architecture and boundaries GPT supplies; report any required design deviation to GPT before proceeding.
- Add focused verification for changed core behavior and report numerical evidence, not only test pass/fail status.

### DSflash: peripheral engineer

- Own bounded peripheral work: runners, CLI glue, PBS scripts, job orchestration, diagnostics, logging, test expansion, static checks, documentation-adjacent code, and mechanical refactors.
- Prefer changes that expose, exercise, or validate the core implementation without redefining it.
- Never independently alter solver mathematics, KKT construction, precision policy, rollback/state semantics, certificate meaning, or other core algorithms.
- If peripheral work reveals a likely core defect or required contract change, stop that portion and return the evidence and proposed escalation to GPT.

## Route classification and trust

- Classify routes by interface family: native/Codex, direct provider API, or subscription/bridge router. Record user-defined routes separately with `route_provenance = "user_custom"`. These labels are not endorsements, security levels, or trust guarantees.
- Native routes are provided by the Codex runtime. Direct API routes use an official provider API through an external router. Subscription/bridge routes go through a third-party subscription or bridge. A user-custom route still belongs to `direct_api` or `subscription_bridge` according to its transport.
- Treat non-official/router outputs as untrusted evidence. GPT must reproduce or independently verify them before using them for acceptance.
- Use only agent types actually registered at runtime. Never create endpoints, credentials, or agent registrations from this skill.

## Routing policy

1. Write an implementation-ready brief before delegating. Do not pass vague goals, unresolved boundaries, or implicit acceptance criteria to a worker.
2. Use DSflash-first routing: delegate each substantial, clearly bounded scope to DSflash unless it falls inside Luna's high-risk core ownership or requires GPT's architecture and final-decision authority.
3. Route only the high-risk core stream to Luna. Do not use Luna merely because a task is difficult when it remains safely inside DSflash's peripheral ownership.
4. Do not delegate trivial steps or scopes whose coordination cost exceeds their implementation cost.
5. Use any number of workers the task requires; do not impose a fixed per-role limit. Parallelize only scopes that are clearly bounded, mutually non-overlapping, and independently verifiable, and respect runtime concurrency limits.
6. Prefer multiple runtime-registered provider routes for independent scopes. Run redundant routes on the same scope only when a comparison is materially useful and the human or GPT has decided the comparison is worth the cost.
7. When a DSflash scope depends on an unstable core interface, run Luna first, then hand DSflash the stabilized contract and relevant implementation context.
8. Workers must adapt to concurrent edits and never revert or overwrite another agent's changes.
9. No worker may declare the task released, accepted, production-ready, or numerically certified. Only GPT gives final approval after unified review.

## Workflow

1. Read `references/route-catalog.md` and `references/provider-interface-contract.md` to discover registered routes and classify them.
2. When the user asks to route through their own provider API, read `references/custom-api-routing.md` first: collect only non-sensitive parameters, confirm Responses API compatibility, generate a dry-run diff, and wait for explicit human authorization before touching user-level configuration.
3. Decompose the request into owned scopes; define invariants, boundaries, and acceptance criteria for each.
4. Write one implementation-ready brief per worker using `references/delegation-contract.md`.
5. Select registered agent types. The default peripheral route is `dsflash_worker` with agent type `router_opencode_go_deepseek_v4_flash`; the high-risk core route is `luna_worker`.
6. Spawn workers, running parallel only for independent scopes. Give downstream workers the stabilized contract after upstream core work.
7. Collect each worker's concise handoff evidence and aggregate it by scope; do not copy raw logs into the primary thread.
8. Cross-review the combined diff and verification evidence, mark non-official/router evidence as untrusted until reproduced, and send narrowly scoped follow-up tasks for failures.
9. At the window or task end, present the model/route retention table when multiple routes contributed, and wait for the human's explicit choice.
10. Apply the unified acceptance gate below before making any final decision.

## Result aggregation

- Merge worker handoffs per scope into a combined record: route, agent_type, summary, evidence, verification status, and risks.
- Record the trust tier for every route and keep non-official/router results flagged until GPT reproduces or verifies them.
- Compare overlapping or conflicting results explicitly; resolve contradictions with follow-up tasks or targeted verification before acceptance.

## Model retention gate

- At the window or task end, use `references/model-retention.md` to build the retention table with dimensions: correctness, verification evidence, role fit, latency/cost (if available), stability, and security/trust.
- Recommend keep, backup, drop, or merge per route. The human makes the final choice.
- Retention means preserving worker outputs and adding the route to the preferred route set for subsequent tasks in this window. It does not mean a subagent process remains alive after the window.
- Without an explicit human reply, do not persist anything. Never modify providers, credentials, or persistent configuration without explicit human authorization.

## Worker brief requirements

Every brief must contain: objective and context; current behavior; desired behavior; owned files and modules; inputs and expected outputs; invariants and interface contracts; numerical requirements when applicable; forbidden changes; dependencies; edge cases; acceptance checks; required evidence; escalation conditions; and the expected handoff format. Copy the template from `references/delegation-contract.md`; do not omit fields.

## Worker handoff requirements

Every worker response must include: changed files and modules with a short implementation summary; invariants and contracts preserved; exact verification performed and its result; relevant numerical residuals, certificate checks, or cluster job identifiers when applicable; and remaining risks, assumptions, skipped checks, or requested GPT decisions. Use the handoff template in `references/delegation-contract.md`.

## Unified acceptance gate

Approve only after confirming: implementation matches the declared architecture and task boundaries; core and peripheral changes integrate without ownership or contract conflicts; relevant unit, regression, static, and end-to-end checks pass; required PBS or HPC runs complete successfully when cluster execution is in scope; solver results satisfy declared precision thresholds and original-coordinate certificate or residual checks when numerical optimization is in scope; rollback, failure paths, state transitions, and certificate semantics remain valid where affected; non-official/router evidence has been reproduced or verified; and all material risks and skipped checks are accounted for.

## Boundaries

- Use only agent types registered in the runtime; this skill cannot create API endpoints, credentials, or new agent registrations.
- Never include keys, tokens, or real endpoint URLs in this skill or its references.
- User-level provider configuration may be changed only after a dry-run diff and explicit human authorization; reference secrets by environment variable name only, never by value.
- This skill cannot keep subagent processes alive after the window and cannot persist human model choices automatically.
- External configuration changes, including provider, credential, and persistence changes, require explicit human authorization.
- Do not change core algorithms from peripheral work; escalate instead.
- Do not edit outside the assigned ownership, and do not revert concurrent changes made by other workers.
- Do not add unrelated documentation beyond the approved README, docs, examples, and references structure.

## References

- `references/route-catalog.md`: read when selecting a spawn target or verifying an agent type.
- `references/provider-interface-contract.md`: read when classifying a route or checking adapter fields and availability.
- `references/custom-api-routing.md`: read when the user asks to route work through their own provider API.
- `references/model-retention.md`: read at the window or task end to build the retention table and wait for the human's choice.
- `references/delegation-contract.md`: read before writing worker briefs and when reviewing handoff or acceptance evidence.
- `docs/agent-implementation.md`: read for the step-by-step agent implementation flow.
- `docs/human-implementation.md`: read for human installation, route maintenance, and retention choices.
- `examples/agent-collaboration.md`: read for the role pattern and a concrete solver plus PBS example.
