# Human Implementation Guide

This guide is for humans installing the skill and maintaining routes. Providers, routers, credentials, and persistent configuration all live outside the skill.

## 1. Install the skill

- Place the whole skill directory where the runtime loads skills, for example `~/.codex/skills/codex-agent-routing-skill/`.
- Confirm the directory contains `SKILL.md` and `agents/openai.yaml`, plus the references, docs, examples, README, and LICENSE files.
- Restart or reload the runtime if the skill list does not pick it up.

## 2. Register providers and routers externally

- The skill only orchestrates agent types already registered in the runtime.
- Register official/native agent types through the runtime. Register direct API and subscription/bridge routes through your external router tooling.
- Do not put real endpoints, keys, tokens, or account identifiers in the skill or repository. Use placeholders in examples. An authorized private endpoint belongs only in user-level configuration; the secret value remains outside configuration and is injected through the named environment variable.
- Verify readiness through the router's own status command before a task needs the route.

## 3. Configure a custom provider and subagent role

- Confirm the provider is Responses API-compatible. If it only exposes Chat Completions, use an external adapter or router; do not configure it as directly compatible.
- Keep provider definitions in the user-level `~/.codex/config.toml`, not in project `.codex/config.toml`, which ignores provider and auth redirection entries.
- Add `[model_providers.<id>]` with `name`, `base_url`, `env_key`, and `wire_api = "responses"` as needed. Use placeholders such as `my_provider`, `https://provider.example.invalid/v1`, `MY_PROVIDER_API_KEY`, and `<provider-model-id>` until the real values are decided.
- Set the secret only through the environment variable named by `env_key`, using an OS secret manager or controlled environment injection. Never write the value into config, prompts, logs, or Git.
- Declare a custom role with user-level `[agents.<name>]` containing `description` and `config_file`, then create the role file with `model` and `model_provider`, for example `~/.codex/agents/my-api-worker.toml`.
- Have Codex show a dry-run diff first. Approve explicitly before any user-level config is written, and make sure the affected config is backed up first.
- Run readiness testing after configuration; only a route that loads, registers, and passes readiness may be added to `references/route-catalog.md`. See `references/custom-api-routing.md` for rollback and deletion steps.

## 4. Maintain the route catalog

- Update `references/route-catalog.md` whenever the runtime registry changes: new agent types, removed types, or renamed routes.
- Keep the exact `agent_type`, `task_name`, `friendly_name`, interface family, route provenance, and model route fields.
- Treat official/native versus non-official/router as an interface classification, not an endorsement or security guarantee.
- Never commit credentials, tokens, or endpoint URLs into the catalog.

## 5. Run a task

- Ask Codex to use the skill when the task needs decomposition, multi-agent routing, parallel work, or unified acceptance.
- The agent will classify routes, write briefs, spawn workers, aggregate evidence, and run the acceptance gate.
- Review the evidence the agent returns, especially reproduced commands and numerical residuals.

## 6. Make the retention choice

- At the window or task end, read the retention table: correctness, verification evidence, role fit, latency/cost (if available), stability, and security/trust.
- Choose keep, backup, drop, or merge per route.
- Reply explicitly. If you do not reply, nothing is persisted and no preferred route set is updated.
- Keep means outputs are preserved and the route is preferred for subsequent tasks in the current window. It does not keep a subagent process alive.
- Provider, credential, or persistent-configuration changes need a separate explicit approval; the skill never makes them by itself.

## 7. Safety checklist

- Skill files contain no real endpoint URLs, keys, or tokens; documentation uses only reserved `.invalid` placeholders.
- The skill never creates agent registrations, endpoints, or credentials.
- The skill never installs a router and never modifies router-managed registry files.
- Custom provider config contains the authorized private endpoint and only the environment variable name, never the secret value. Public previews use placeholder endpoints; the actual secret lives in the environment.
- Dry-run diffs precede every user-level config change, and rollback steps are defined before the change is applied.
- Worker outputs from non-official routes are treated as untrusted until reproduced.
