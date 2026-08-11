# Codex Agent Routing Skill

Reusable skill for decomposing non-trivial work, delegating bounded scopes across GPT, Luna, and DSflash workers, and choosing among runtime-registered official, direct-API, and subscription/bridge router agent types. GPT remains the architect and sole final reviewer; workers return summaries and verification evidence; the human makes the final route retention choice at the window or task end.

## Capabilities

- Task decomposition with exact ownership boundaries and invariants.
- Multi-provider parallel routing across runtime-registered agent types.
- Interface classification: native, direct provider API, and subscription/bridge router.
- DSflash-first peripheral routing and Luna-owned high-risk core routing.
- Result aggregation, cross-review, and unified acceptance by GPT.
- Model/route retention gate with keep, backup, drop, and merge recommendations.

## Limitations

- Only spawns agent types already registered at runtime.
- Does not create endpoints, credentials, agent registrations, or router installs.
- Cannot keep subagent processes alive after the window.
- Does not persist model choices without an explicit human reply.
- Official/native versus non-official/router is an interface classification, not endorsement, security, or billing status.

## Structure

```text
codex-agent-routing-skill/
├── SKILL.md
├── README.md
├── LICENSE
├── agents/openai.yaml
├── references/
│   ├── route-catalog.md
│   ├── provider-interface-contract.md
│   ├── model-retention.md
│   └── delegation-contract.md
├── docs/
│   ├── agent-implementation.md
│   └── human-implementation.md
└── examples/
    └── agent-collaboration.md
```

## Quick start

1. Place the skill directory where the runtime loads skills, for example `~/.codex/skills/codex-agent-routing-skill/`.
2. Register the agent types your project needs through the runtime or an external router; keep credentials and endpoints outside this repository.
3. Update `references/route-catalog.md` to match the runtime registry.
4. Ask Codex to use the skill for the task. The agent classifies routes, writes worker briefs, spawns workers in parallel where scopes are independent, aggregates handoffs, and runs the unified acceptance gate.
5. At the window or task end, review the retention table and reply with keep, backup, drop, or merge. No reply means no persistence.

## Security boundary

This skill never contains or requests real endpoint URLs, keys, or tokens. Non-official/router outputs are untrusted evidence until GPT reproduces or independently verifies them. Provider, credential, and persistent-configuration changes require explicit human authorization.

## Model retention flow

Worker outputs are collected per route and scored on correctness, verification evidence, role fit, latency/cost (if available), stability, and security/trust. GPT recommends keep, backup, drop, or merge; the human decides. Keeping a route preserves its outputs and adds the route to the preferred route set for subsequent tasks in the current window.

## License

MIT. See [LICENSE](LICENSE).
