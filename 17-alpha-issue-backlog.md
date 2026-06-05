# Alpha 本地 Issue Backlog

本文档把 [16-alpha-implementation-plan.md](16-alpha-implementation-plan.md) 中的 A0-A12 转成本地可领取任务。

这些任务尚未发布到远端 issue tracker。进入开发前，可按本文档复制为正式 issue 或 sprint item。

## 1. 拆分原则

- 每个任务都应能独立验收。
- 优先按可演示路径推进，而不是按系统层横切。
- 阻塞任务先做，依赖任务后做。
- Alpha 不纳入 Beta 能力。

## 2. Backlog

### A0 冻结样例校验器

类型：AFK

Blocked by：None - can start immediately

What to build：

实现一个开发前校验入口，解析 [15-alpha-v0-fixtures.md](15-alpha-v0-fixtures.md) 中所有 JSON 样例，并校验 Alpha 主链路的关键 ID 闭合。

Acceptance criteria：

- [ ] 能解析 `15-alpha-v0-fixtures.md` 中全部 JSON 样例。
- [ ] 校验 `product_research_ai_km_001`、`package_research_ai_km_001_v1`、`task_research_ai_km_001` 闭合。
- [ ] 校验失败时能指出失败的 JSON block 或缺失 ID。

### A1 供端能力目录

类型：AFK

Blocked by：A0

What to build：

让产端可以查询 Alpha 报告线所需的工具能力、额度和调用策略。

Acceptance criteria：

- [ ] 可查询 `search.web`、`text.generate`、`report.export`、`content.moderate`。
- [ ] 可返回 `ToolCapability`、`ResourceQuota`、`ToolInvocationPolicy`。
- [ ] `research_report` 只能拿到 Alpha 允许的能力。

### A2 供端工具调用与 fallback

类型：AFK

Blocked by：A1

What to build：

支持工具节点调用供端能力，记录 provider、成本、错误和输出引用，并在默认 provider 失败时执行 fallback。

Acceptance criteria：

- [ ] 工具调用记录绑定 `production_task_id` 和 `node_execution_id`。
- [ ] 默认 provider 超时后能走 fallback provider 或 mock fallback。
- [ ] 产端只依赖最终 `output_ref`。

### A3 资产引用服务

类型：AFK

Blocked by：A0

What to build：

支持 Alpha 使用的 `artifact://` 和 `asset://` 受控解析，区分中间产物和正式商品资产。

Acceptance criteria：

- [ ] `artifact://` 只供产端和运营审核视图使用。
- [ ] `asset://` 可用于正式商品包预览和下载。
- [ ] 销端不能把中间 `artifact://` 作为展示或下载依赖。

### A4 生产任务运行骨架

类型：AFK

Blocked by：A0

What to build：

支持创建 Alpha 报告生产任务、保存任务状态、记录节点执行，并保留失败和恢复入口。

Acceptance criteria：

- [ ] 可创建 `research_report` 生产任务。
- [ ] 支持 `created`、`running`、`waiting_for_human`、`completed`、`failed`。
- [ ] 每个节点执行生成 `NodeExecution`。

### A5 报告线 Workflow Runner

类型：AFK

Blocked by：A1、A4

What to build：

加载 `workflow_research_report_v0`，按冻结模板执行节点，并能在人工节点暂停。

Acceptance criteria：

- [ ] 能从 `input_intake` 执行到 `outline_review`。
- [ ] 到 `outline_review` 时任务进入 `waiting_for_human`。
- [ ] 当前节点和等待原因可查询。

### A6 质量门禁与重跑

类型：AFK

Blocked by：A5

What to build：

支持质量门禁输出 `pass`、`manual_review`、`rerun`、`fail`，并按 `target_node_for_rerun` 重跑。

Acceptance criteria：

- [ ] `fact_check` 可输出 `rerun`。
- [ ] `rerun` 记录 `reason` 和 `target_node_for_rerun`。
- [ ] 被重跑节点生成新的 `NodeExecution`，不覆盖旧产物。

### A7 人工干预恢复

类型：AFK

Blocked by：A5、A6

What to build：

支持 creator 和 operator 在人工节点执行允许动作，并从恢复节点继续流水线。

Acceptance criteria：

- [ ] creator 可执行 `approve`、`edit`、`choose`、`rerun`。
- [ ] operator 可执行 `approve`、`edit`、`choose`、`rerun`、`reject`、`override`。
- [ ] `edit` 生成新的 `output_ref`。
- [ ] `override` 必须记录 `risk_note`，且仍进入最终质量门禁。

### A8 商品包生成与固化

类型：AFK

Blocked by：A3、A6、A7

What to build：

支持候选商品包生成、最终质量门禁和正式 `ProductPackage` 固化。

Acceptance criteria：

- [ ] 生成候选包时资产范围为 `candidate_package`。
- [ ] 最终质量门禁通过后固化正式 `ProductPackage`。
- [ ] 正式商品包状态为 `ready_for_listing`。
- [ ] 生产任务完成后进入 `completed`。

### A9 销端商品接入与展示

类型：AFK

Blocked by：A8

What to build：

销端接入正式 `ProductPackage`，创建 `Listing`，并提供列表和详情展示数据。

Acceptance criteria：

- [ ] 只接收 `ready_for_listing` 商品包。
- [ ] 创建 `listing_research_ai_km_001_v1`。
- [ ] 商品详情只依赖 `ProductPackage` 和 `Listing`。

### A10 模拟购买、权益和下载

类型：AFK

Blocked by：A9

What to build：

支持用户模拟购买或免费下载，生成订单、权益和下载记录。

Acceptance criteria：

- [ ] 生成 `OrderRecord.payment_status = simulated_paid`。
- [ ] 生成与订单绑定的 `Entitlement`。
- [ ] 生成绑定 `asset_report_ai_km_pdf_001` 的 `DownloadRecord`。
- [ ] 订单、权益、下载记录绑定同一 `package_id` 和 `version`。

### A11 MarketFeedback 回流

类型：AFK

Blocked by：A10

What to build：

销端聚合浏览、下载、购买、评分和评论数据，生成并回传绑定版本的 `MarketFeedback`。

Acceptance criteria：

- [ ] 生成 `feedback_research_ai_km_001_2026_06_05`。
- [ ] feedback 绑定 `product_id`、`package_id` 和 `version`。
- [ ] feedback 不直接修改 `WorkflowTemplate` 或触发优化任务。

### A12 Alpha 验收脚本

类型：AFK

Blocked by：A1-A11

What to build：

提供一个 Alpha 验收入口，按测试矩阵串起主闭环和异常场景。

脚本规格见 [18-alpha-acceptance-script-spec.md](18-alpha-acceptance-script-spec.md)。

Acceptance criteria：

- [ ] 覆盖 T1 报告主闭环。
- [ ] 覆盖 T2 质量门禁重跑。
- [ ] 覆盖 T3 人工编辑恢复。
- [ ] 覆盖 T4 运营覆盖。
- [ ] 覆盖 T5 工具 fallback。
- [ ] 覆盖 T6 模拟购买下载。
- [ ] 覆盖 T7 反馈回流。
- [ ] 覆盖 S8 边界守卫。
- [ ] S0-S8 均输出可追踪证据。

## 3. 发布顺序

建议发布正式 issue 时按以下顺序：

```text
A0
A1
A3
A4
A2
A5
A6
A7
A8
A9
A10
A11
A12
```

如果团队需要并行，可以在 A0 完成后同时启动 A1、A3、A4。
