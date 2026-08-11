# Model and Route Retention Protocol

Read this file at the window or task end when multiple routes contributed. It defines the exact meaning of retention and the human-facing protocol for keeping, backing up, dropping, or merging routes.

## Retention semantics

- **Keep**: preserve the worker's outputs and add the route to the preferred route set for subsequent tasks in this window.
- **Backup**: preserve artifacts and evidence without adding the route to the preferred route set.
- **Drop**: do not use the route again and remove it from the preferred route set.
- **Merge**: combine outputs or evidence from two routes into one verified result, then apply keep or backup to the merged result.

Retention does **not** keep a subagent process alive. Subagents are short-lived; after the window or task ends, only outputs and the preferred-route preference remain. The skill never persists the human's choice automatically, and no provider, credential, or persistent configuration is changed without explicit human authorization.

## Evaluation dimensions

| Dimension | Question | Evidence source |
| --- | --- | --- |
| Correctness | Did the route's output satisfy the declared acceptance criteria? | Tests, checks, diff review |
| Verification evidence | Can the evidence be reproduced exactly, including residuals and certificates? | Commands, logs, artifacts, cluster job IDs |
| Role fit | Did the route stay inside its assigned ownership? | Brief vs. handoff comparison |
| Latency/cost | Was latency and cost acceptable, when measurable? | Runtime observability, billing surfaces if available |
| Stability | Did the route behave consistently across runs and edge cases? | Repeated runs, failure rates |
| Security/trust | Was the output trustworthy and was the route's trust tier honored? | Registry classification, verification status, secrets handling |

## Recommendation rules

- **Keep**: acceptance criteria pass, evidence is reproducible, role fit is clean, behavior is stable, and the trust tier was honored (untrusted-router output was independently verified).
- **Backup**: output is useful but not yet fully verified, or the route is a candidate for future comparison.
- **Drop**: checks fail, evidence cannot be reproduced, the route repeatedly violates ownership, or untrusted output was not verified.
- **Merge**: routes show complementary strengths or partially overlapping outputs; merge artifacts and evidence, then re-verify the combined result.

## End-of-window protocol

1. Aggregate all handoffs and group them by route.
2. Score each route against the six dimensions above.
3. GPT records one recommendation per route: keep, backup, drop, or merge.
4. Present the display table to the human and wait for an explicit reply.
5. Apply only the human's explicit choice. No reply means no persistence.
6. A keep or merged-keep choice adds the route to the preferred route set for subsequent tasks in the current window.
7. Any provider, credential, or persistent-configuration change requires separate explicit human authorization.

## Display table template

```text
Route retention at window end

| Route | agent_type | Correctness | Verification | Role fit | Latency/cost | Stability | Security/trust | Recommendation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DSflash_worker | router_opencode_go_deepseek_v4_flash | pass | reproducible | clean | acceptable | stable | verified | keep |
| DeepSeek V4 Pro API | router_deepseek_deepseek_v4_pro | pass | reproducible | clean | not measured | stable | verified | backup |
| GLM 5.2 bridge | router_opencode_go_glm_5_2 | partial | failed rerun | clean | acceptable | unstable | unverified | drop |

Human decision: keep / backup / drop / merge
```

Fill the table with the actual routes used and the evidence collected. Keep the recommendation column conservative: when evidence is missing, prefer backup or drop over keep.
