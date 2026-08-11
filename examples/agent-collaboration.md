# Agent Collaboration Example

This file records the collaboration model behind the skill and gives a concrete solver plus PBS scenario.

## Original collaboration model

以下是用户最初写下的协作条目：

- GPT（架构师/Reviewer）：定义架构、数值不变量、任务边界和验收标准；负责拆分任务及最终决策。
- Luna worker（核心 Engineer）：实现高风险核心修改，包括求解路径、KKT、精度、rollback、状态与证书语义。
- DeepSeek worker（外围 Engineer）：并行处理 runner、PBS、diagnostics、测试扩展、静态检查和机械性重构，不独立改变核心算法。
- 最终验收：所有实现返回修改摘要与验证证据；GPT 根据测试、集群运行和原坐标证书统一审查。任何 worker 不得自行宣布发布通过。

后续演进规则：GPT 先把任务写成 implementation-ready brief；所有不涉及核心算法或架构决策的 substantial scope 优先交给 DSflash。worker 数量按任务需要决定，不设固定上限；只对边界清晰、互不重叠、可独立验证的 scope 并行。

## Generalized execution model

- GPT: define architecture, invariants, boundaries, and acceptance criteria; split tasks; decide parallel or sequential; review all evidence; make the only final release decision.
- Luna: own solver paths, KKT systems, precision, rollback, mutable state, recovery, and certificate semantics; return numerical evidence.
- DSflash/DeepSeek: own runners, CLI glue, PBS scripts, diagnostics, logging, test expansion, static checks, documentation-adjacent code, and mechanical refactors; never change core algorithms.
- Every worker returns a summary, preserved invariants, exact verification evidence, and risks or skipped items.
- No worker announces release acceptance; GPT reviews tests, cluster runs, and original-coordinate certificates or residuals before approving.

## Concrete example: conic solver rollback on PBS

Scenario: add rollback behavior to a conic solver and validate it on a PBS cluster.

1. GPT defines the contract: solver entry point, KKT construction, original-coordinate certificate, residual threshold, rollback state semantics, and owned file boundaries.
2. GPT writes a Luna brief for the core stream: solver path, KKT system, precision policy, rollback/recovery state, certificate meaning. Acceptance requires residual and certificate evidence.
3. GPT writes a DSflash brief for the peripheral stream: runner, CLI flag, PBS job scripts, diagnostics, logging, test expansion, and static checks. Acceptance requires test output, static-check output, and a PBS job identifier.
4. Execution order: Luna stabilizes the core contract first; DSflash then runs in parallel after receiving the stabilized interface.
5. Handoffs: Luna returns changed files, preserved invariants, residual/certificate values, and risks. DSflash returns changed files, tests, static-check results, and cluster job identifiers.
6. GPT reviews the combined diff, reruns tests, submits or inspects the PBS job, and checks the original-coordinate certificate and residual thresholds.
7. GPT decides whether to approve, send follow-up tasks, or reject. Neither worker declares the release passed.

## Generic PBS example placeholder

Use placeholders only in examples; never commit real account, endpoint, or credential values.

```text
#PBS -N solver-rollback-<scope>
#PBS -l select=1:ncpus=16:mem=64gb
#PBS -l walltime=02:00:00
cd $PBS_O_WORKDIR
source <environment-setup-command>
python <solver-test-entrypoint> --residual-threshold <threshold> --certificate-output <path>
```

Replace `<...>` entries with project-specific values during execution, and keep the real values outside version control.
