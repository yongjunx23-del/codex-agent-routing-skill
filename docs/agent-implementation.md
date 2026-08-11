# Agent Implementation Guide

This guide is for Codex or any agent using the skill. Follow it end to end: discovery, classification, briefing, concurrent spawn, handoff collection, cross-review, retention, and final acceptance.

## 1. Discover registered routes

- Inspect the runtime agent registry only. On local Codex installs this is typically the agents directory, for example `~/.codex/agents/*.toml`; router tooling may also expose a readiness listing.
- Record for each route: `agent_type`, `task_name`, `friendly_name`, and source. Do not guess or invent values.
- If a route you need is absent, stop and report the gap. Do not create an agent type from the skill.

## 2. Classify routes

- Apply `references/provider-interface-contract.md`: assign `native`, `direct_api`, or `subscription_bridge`.
- Assign the trust tier and evidence requirement. Subscription/bridge output starts as untrusted evidence.
- Keep endpoint and credential details out of the skill. Only confirm externally that the router reports the provider ready.

## 3. Decompose and write briefs

- Decompose the task into owned scopes with non-overlapping boundaries.
- For each scope, define invariants, forbidden changes, acceptance checks, and required evidence.
- Write one implementation-ready brief per worker using `references/delegation-contract.md`.
- Default routing: peripheral scope to `dsflash_worker` with agent type `router_opencode_go_deepseek_v4_flash`; high-risk core scope to `luna_worker`.

## 4. Select routes and parallelize

- Prefer multiple registered provider routes for independent scopes.
- Use redundant routes on the same scope only when comparison is materially useful and explicitly decided.
- Respect runtime concurrency limits. Parallelize only clearly bounded, non-overlapping, independently verifiable scopes.
- Sequence work that depends on an unstable core interface: run Luna first, then give DSflash the stabilized contract.

## 5. Spawn concurrently

- Spawn each worker with its exact `task_name` and `agent_type` from the catalog.
- Include the full brief in the spawn message; never send vague objectives.
- Tell every worker to adapt to concurrent edits and never revert changes made by other agents.

## 6. Collect handoffs and aggregate

- Require the handoff structure from `references/delegation-contract.md`.
- Aggregate by scope: route, agent_type, summary, changed files, invariants preserved, verification evidence, risks, and decisions requested.
- Record the trust tier for each result and flag untrusted-router evidence.

## 7. Cross-review

- Compare overlapping results and resolve conflicts with follow-up tasks or targeted checks.
- Reproduce every claimed command, test, residual, or certificate, especially for untrusted-router evidence.
- Mark evidence verified only when the reproduction succeeds with the declared thresholds.

## 8. Build the retention table

- At the window or task end, score each contributing route against `references/model-retention.md`.
- Use the dimensions: correctness, verification evidence, role fit, latency/cost (if available), stability, and security/trust.
- Present keep/backup/drop/merge recommendations and wait for the human's explicit reply. No reply means no persistence.

## 9. GPT final acceptance

- Approve only after the unified acceptance gate passes and the human retention choice is recorded when required.
- The final record must state: what changed, what was verified, which routes are preferred, and which risks or skipped checks remain.
- No worker announces release, acceptance, production readiness, or numerical certification; that decision belongs to GPT and the human.
