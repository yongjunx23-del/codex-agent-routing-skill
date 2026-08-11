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

## 3. Handle custom provider requests

- When the user asks to route through their own API, read `references/custom-api-routing.md`.
- Collect only non-sensitive parameters: provider display name, model ID, key environment variable name, Responses API compatibility, and purpose or role. Treat a private base URL as sensitive metadata: use a `.invalid` placeholder in chat and previews, and write the exact value only to user-level configuration after explicit authorization. Never ask for or echo secret values.
- Generate a dry-run diff for the user-level `~/.codex/config.toml` provider block, optional `$CODEX_HOME/<profile>.config.toml`, and the subagent role file; wait for explicit human authorization before writing anything.
- Apply only after approval and after backing up the affected files. Run readiness checks, then register the exact runtime-registered `agent_type` in `references/route-catalog.md` only after verification passes; roll back on failure.
- Never write user provider config or role files into Git, and never let command output or logs contain credentials.

## 4. Decompose and write briefs

- Decompose the task into owned scopes with non-overlapping boundaries.
- For each scope, define invariants, forbidden changes, acceptance checks, and required evidence.
- Write one implementation-ready brief per worker using `references/delegation-contract.md`.
- Default routing: peripheral scope to `dsflash_worker` with agent type `router_opencode_go_deepseek_v4_flash`; high-risk core scope to `luna_worker`.

## 5. Select routes and parallelize

- Prefer multiple registered provider routes for independent scopes.
- Use redundant routes on the same scope only when comparison is materially useful and explicitly decided.
- Respect runtime concurrency limits. Parallelize only clearly bounded, non-overlapping, independently verifiable scopes.
- Sequence work that depends on an unstable core interface: run Luna first, then give DSflash the stabilized contract.

## 6. Spawn concurrently

- Spawn each worker with its exact `task_name` and `agent_type` from the catalog.
- Include the full brief in the spawn message; never send vague objectives.
- Tell every worker to adapt to concurrent edits and never revert changes made by other agents.

## 7. Collect handoffs and aggregate

- Require the handoff structure from `references/delegation-contract.md`.
- Aggregate by scope: route, agent_type, summary, changed files, invariants preserved, verification evidence, risks, and decisions requested.
- Record the trust tier for each result and flag untrusted-router evidence.

## 8. Cross-review

- Compare overlapping results and resolve conflicts with follow-up tasks or targeted checks.
- Reproduce every claimed command, test, residual, or certificate, especially for untrusted-router evidence.
- Mark evidence verified only when the reproduction succeeds with the declared thresholds.

## 9. Build the retention table

- At the window or task end, score each contributing route against `references/model-retention.md`.
- Use the dimensions: correctness, verification evidence, role fit, latency/cost (if available), stability, and security/trust.
- Present keep/backup/drop/merge recommendations and wait for the human's explicit reply. No reply means no persistence.

## 10. GPT final acceptance

- Approve only after the unified acceptance gate passes and the human retention choice is recorded when required.
- The final record must state: what changed, what was verified, which routes are preferred, and which risks or skipped checks remain.
- No worker announces release, acceptance, production readiness, or numerical certification; that decision belongs to GPT and the human.
