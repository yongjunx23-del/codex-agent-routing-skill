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

## Copy-paste prompt: let Codex install it

Paste the following prompt into Codex. It tells the agent to use the bundled Skill Installer, install from this public repository, validate the result, and avoid overwriting an existing installation without approval.

```text
请帮我安装这个 Codex skill：
https://github.com/yongjunx23-del/codex-agent-routing-skill

要求：
1. 先读取并遵循本机 `skill-installer` 的 SKILL.md；优先使用它自带的 `install-skill-from-github.py`，不要手工复制零散文件。
2. 从仓库 `yongjunx23-del/codex-agent-routing-skill` 的 `main` 分支安装仓库根目录，技能名固定为 `codex-agent-routing-skill`。
3. 安装到 `$CODEX_HOME/skills/codex-agent-routing-skill`；如果未设置 `CODEX_HOME`，使用 `~/.codex/skills/codex-agent-routing-skill`。
4. 如果目标目录已经存在，不要覆盖或删除；先检查现有版本并向我说明需要更新还是保留。
5. 安装后确认 `SKILL.md` 和 `agents/openai.yaml` 存在，并使用 skill-creator 的 `quick_validate.py` 做结构验证（若本机可用）。
6. 不要创建、修改或输出任何 API endpoint、key、token 或 provider 凭据；这些接口仍由外部 runtime/router 注册。
7. 最后告诉我实际安装路径、验证结果，以及该技能会从下一轮对话开始可用。除非遇到已有目录冲突或需要额外权限，否则直接完成安装，不要只给我安装步骤。
```

## Security boundary

This skill never contains or requests real endpoint URLs, keys, or tokens. Non-official/router outputs are untrusted evidence until GPT reproduces or independently verifies them. Provider, credential, and persistent-configuration changes require explicit human authorization.

## Model retention flow

Worker outputs are collected per route and scored on correctness, verification evidence, role fit, latency/cost (if available), stability, and security/trust. GPT recommends keep, backup, drop, or merge; the human decides. Keeping a route preserves its outputs and adds the route to the preferred route set for subsequent tasks in the current window.

## License

MIT. See [LICENSE](LICENSE).
