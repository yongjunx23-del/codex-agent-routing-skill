# Provider and Route Interface Contract

Read this file before classifying a route, writing an adapter record, or checking whether a route is usable. It defines the common fields every provider/route adapter must expose and the availability rules the skill must enforce.

## Common adapter fields

Every route used by this skill must be describable with these fields. Do not add fields that require endpoint URLs or credentials.

| Field | Required | Meaning |
| --- | --- | --- |
| `agent_type` | yes | Exact value registered in the runtime; the only value spawnable by this skill |
| `task_name` | yes | Identifier used when spawning the agent |
| `friendly_name` | yes | Human-facing label used in briefs and retention tables |
| `interface_family` | yes | `native`, `direct_api`, or `subscription_bridge` |
| `route_provenance` | yes | `runtime_builtin`, `router_registered`, or `user_custom`; ownership metadata, not a transport classification |
| `model_route` | yes | Provider/model identifier used by the external router, for reference only |
| `registry_source` | yes | Where the type is registered, such as runtime built-in or a registry file |
| `availability_check` | yes | How to verify the route is present without creating anything |
| `trust_tier` | yes | `native`, `provider_official`, or `untrusted_router` |
| `evidence_requirement` | yes | Verification standard for outputs from this route |

## Interface families

### Codex/OpenAI native

- Includes runtime-provided types such as `default`, `explorer`, `worker`, and `luna_worker`.
- No external router, endpoint, or credential is needed.
- Availability: present in the runtime registry.
- Trust tier: native. Evidence still passes the unified acceptance gate.

### Direct API router

- Includes routes such as the DeepSeek direct API family, where an official provider API is reached through an external router.
- Requires externally configured credentials and billing outside this skill.
- Availability: registered in the runtime registry, and the external router reports the provider configured.
- Trust tier: provider_official. Outputs still require normal implementation verification and can be cross-checked.

### Subscription/bridge router

- Includes routes such as the OpenCode Go family, where a third-party subscription or bridge fronts the model.
- Requires externally configured subscription access or bridge credentials outside this skill.
- Availability: registered in the runtime registry, and the external router reports the bridge ready.
- Trust tier: untrusted_router. Outputs are non-official evidence and must be reproduced or independently verified by GPT before acceptance.

### User-custom provider provenance

- Includes user-defined providers and custom subagent roles registered through user-level configuration, as described in `custom-api-routing.md`.
- Requires a Responses-compatible endpoint or an external adapter/router for Chat Completions-only APIs, plus externally injected credentials.
- Availability: registered in the runtime registry and passing readiness testing; being user-configured is not itself an availability guarantee.
- Set `route_provenance = "user_custom"`. Keep `interface_family` as `direct_api` when it uses an official provider API, or `subscription_bridge` when a third-party subscription or bridge is involved.
- The trust tier follows the mapped family, and user configuration never lowers the evidence requirement.

## Availability checks

- Check only the runtime agent registry and the external router's own readiness state. Never create, edit, or guess agent types.
- Registry presence is necessary but not sufficient: a spawn that fails to start or errors immediately means the route is unavailable for this task.
- If a route is missing, stop that portion and report the gap to GPT or the human; do not fall back to inventing an agent_type.
- Resolve endpoint and credential status exclusively from external configuration. This skill never stores or supplies them.

## Trust and evidence policy

- Native evidence: verified through normal tests, static checks, and integration runs.
- Provider-official evidence (direct API): verify commands, results, and numerical artifacts like any implementation; provider origin is not a substitute for reproduction.
- Untrusted-router evidence (subscription/bridge): mark as untrusted in the aggregation record; GPT must reproduce the result or verify it through independent evidence before it can support acceptance.
- User-custom evidence: follows the mapped `direct_api` or `subscription_bridge` policy; custom configuration does not make evidence more trustworthy.
- Role fit still applies: even a verified route must not be used to change core algorithms unless ownership says otherwise.

## Registration boundaries

- This skill orchestrates only runtime-registered agent types.
- It cannot create API endpoints, credentials, or agent registrations, and it cannot install a router.
- Router-managed registry files are owned by external tooling; do not edit them from this skill.
