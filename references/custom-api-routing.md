# Custom API Routing

Read this file whenever a user asks to route work through their own provider API. It covers Responses API compatibility checks, safe user-level configuration, credential injection, subagent roles, readiness testing, catalog registration, and rollback.

## Official reference facts

- Custom providers are defined in the user-level `~/.codex/config.toml`. Project-level `.codex/config.toml` ignores provider and auth redirection entries such as `model_provider` and `model_providers`.
- `[model_providers.<id>]` supports `name`, `base_url`, `env_key`, and optional `wire_api = "responses"`. The current official reference supports only `wire_api = "responses"`.
- Secrets are referenced by `env_key`, which names an environment variable. Never write the secret value into configuration, this skill, prompts, or logs. `experimental_bearer_token` is officially discouraged.
- Custom subagent roles can be declared in user-level `[agents.<name>]` with `description` and `config_file`. The role file sets `model` and `model_provider`, referencing an already-defined provider.
- Only an `agent_type` that the runtime actually loads, registers, and passes readiness testing may be added to `route-catalog.md` or invoked by this skill.
- Official references: [config reference](https://learn.chatgpt.com/docs/config-file/config-reference) and [custom model providers](https://learn.chatgpt.com/docs/config-file/config-advanced#custom-model-providers).

## Preflight: Responses API compatibility

- Direct custom-provider configuration requires a Responses API-compatible endpoint.
- If the provider exposes only Chat Completions, do not pretend it is directly compatible. Require an external adapter or router that translates between interfaces before adding the route.
- Ask the human for a compatibility statement before writing configuration. If compatibility is unknown, require an adapter or a successful readiness test that proves the endpoint accepts Responses API requests.

## Safety templates

Use only placeholders in examples. Replace them with real values during execution, and keep real values outside this repository and outside Git.

```toml
# ~/.codex/config.toml (user-level; placeholders only)
[model_providers.my_provider]
name = "My Provider"
base_url = "https://provider.example.invalid/v1"
env_key = "MY_PROVIDER_API_KEY"
wire_api = "responses"
```

Optional profile or primary model use. In Codex 0.134 and later, a named profile is a separate file under `$CODEX_HOME`, not a `[profiles.*]` table in `config.toml`:

```toml
# ~/.codex/my-provider.config.toml (user-level; optional, placeholders only)
model = "<provider-model-id>"
model_provider = "my_provider"
```

Select it with `codex --profile my-provider`. Keep the provider definition itself in user-level `~/.codex/config.toml`.

Custom subagent role declaration:

```toml
# ~/.codex/config.toml (user-level)
[agents.my_api_worker]
description = "Bounded worker routed through my_provider"
config_file = "agents/my-api-worker.toml"
```

Relative `config_file` paths resolve from the config file that declares the role. For the user-level config above, this points to `~/.codex/agents/my-api-worker.toml`.

```toml
# ~/.codex/agents/my-api-worker.toml
model = "<provider-model-id>"
model_provider = "my_provider"
```

## Credential injection rules

- Store only the environment variable name in `env_key`; the value must come from the environment at runtime.
- Never write key values into configuration, this skill, prompts, logs, diffs, or Git.
- Prefer an OS secret manager or controlled environment injection. Avoid pasting secrets directly into interactive shell history, which may capture the value.
- Command-backed auth is an advanced option. Read the official config-advanced reference before using it, do not combine it with `env_key`, and never log or echo command output that contains credentials.

## Readiness test and failure handling

- After writing or editing user-level configuration, back up the affected config files first.
- Run the runtime's config validation or doctor command when available.
- Spawn `my_api_worker` with a minimal, harmless task and confirm it completes with a valid response.
- If startup or auth fails, inspect only redacted error output. Verify provider id, model id, `env_key`, `base_url`, and Responses API compatibility, then fix and retest.
- Do not print environment variables, request headers, or response bodies that may contain secrets.

## Catalog registration

- Add the route to `references/route-catalog.md` only after readiness passes and only with the exact runtime-registered `agent_type`.
- Record `user_custom` as route provenance, not as an interface family. Set `interface_family` to `direct_api` when it uses an official provider API, or to `subscription_bridge` when a third-party subscription or bridge is involved.
- Assign the corresponding trust tier and keep the evidence requirement unchanged. Being user-configured never lowers the evidence bar.

## Rollback and deletion

- Keep a timestamped backup of the user-level config before any edit.
- To roll back, remove the `[model_providers.<id>]` block, any optional `$CODEX_HOME/<profile>.config.toml` file, the `[agents.<name>]` declaration, and the role file.
- Remove or replace the external environment injection if it was added only for this route.
- Rerun the readiness check and confirm the route no longer loads.
- Never commit the provider config, role file, or any secret-related values to Git.

## Copyable prompt for Codex (Chinese)

把下面这段提示词原样发给 Codex，让它安全地收集参数并完成用户级配置：

```text
请帮我配置一个自定义 API 路由，只使用以下占位符流程：

1. 先向我询问这些非敏感参数：provider 显示名、model ID、密钥环境变量名、Responses API 兼容性、用途/角色。base URL 若属于私有信息，只在我明确授权后直接写入用户级配置，不要在对话、日志或公开 diff 中回显；预览时统一使用 `.invalid` 占位地址。
2. 绝对不要询问或回显密钥值；只使用密钥环境变量名，并通过环境变量注入。
3. 检查目标 API 是否兼容 Responses API；如果只有 Chat Completions，不要假装直接兼容，改为要求外部 adapter/router 转译。
4. 先生成一份 dry-run diff，展示将写入用户级 `~/.codex/config.toml` 的 `[model_providers.<id>]` 和 `[agents.<name>]`、可选的 `$CODEX_HOME/<profile>.config.toml`，以及 `~/.codex/agents/*.toml` 角色文件内容。所有预览使用明确占位符，例如 `https://provider.example.invalid/v1`、`MY_PROVIDER_API_KEY`、`<provider-model-id>`。
5. 在我明确授权之前，不要写入或修改任何用户级配置；得到授权后再执行，并先备份受影响文件。
6. 配置后运行可用的验证/readiness 测试；只有通过后，才把实际注册的 `agent_type` 加入 `references/route-catalog.md`。
7. 若失败，回滚备份并清理相关配置；不要提交这些配置到 Git。
8. 不得把真实 endpoint、密钥、token 或账号凭据提交到 Git 或写入公开日志；不得询问或回显密钥值。私有 endpoint 只可在我明确授权后写入用户级配置，并在对话与验证输出中脱敏。
```
