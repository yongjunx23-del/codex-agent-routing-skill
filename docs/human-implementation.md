# Human Implementation Guide

This guide is for humans installing the skill and maintaining routes. Providers, routers, credentials, and persistent configuration all live outside the skill.

## 1. Install the skill

- Place the whole skill directory where the runtime loads skills, for example `~/.codex/skills/codex-agent-routing-skill/`.
- Confirm the directory contains `SKILL.md` and `agents/openai.yaml`, plus the references, docs, examples, README, and LICENSE files.
- Restart or reload the runtime if the skill list does not pick it up.

## 2. Register providers and routers externally

- The skill only orchestrates agent types already registered in the runtime.
- Register official/native agent types through the runtime. Register direct API and subscription/bridge routes through your external router tooling.
- Do not put endpoints, keys, tokens, or real account identifiers in the skill. Use placeholders such as `<provider-key-here>` only in examples, never in real files.
- Verify readiness through the router's own status command before a task needs the route.

## 3. Maintain the route catalog

- Update `references/route-catalog.md` whenever the runtime registry changes: new agent types, removed types, or renamed routes.
- Keep the exact `agent_type`, `task_name`, `friendly_name`, interface family, and model route fields.
- Treat official/native versus non-official/router as an interface classification, not an endorsement or security guarantee.
- Never commit credentials, tokens, or endpoint URLs into the catalog.

## 4. Run a task

- Ask Codex to use the skill when the task needs decomposition, multi-agent routing, parallel work, or unified acceptance.
- The agent will classify routes, write briefs, spawn workers, aggregate evidence, and run the acceptance gate.
- Review the evidence the agent returns, especially reproduced commands and numerical residuals.

## 5. Make the retention choice

- At the window or task end, read the retention table: correctness, verification evidence, role fit, latency/cost (if available), stability, and security/trust.
- Choose keep, backup, drop, or merge per route.
- Reply explicitly. If you do not reply, nothing is persisted and no preferred route set is updated.
- Keep means outputs are preserved and the route is preferred for subsequent tasks in the current window. It does not keep a subagent process alive.
- Provider, credential, or persistent-configuration changes need a separate explicit approval; the skill never makes them by itself.

## 6. Safety checklist

- Skill files contain no endpoint URLs, keys, or tokens.
- The skill never creates agent registrations, endpoints, or credentials.
- The skill never installs a router and never modifies router-managed registry files.
- Worker outputs from non-official routes are treated as untrusted until reproduced.
