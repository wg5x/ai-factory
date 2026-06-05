# 开发就绪评审

本文档用于在进入开发前做最终收敛。它不替代架构、契约、数据模型、接口边界和验收计划，只汇总第一阶段能否开工的判断口径。

## 1. 开发前结论

第一阶段按两层推进：

```text
Alpha：报告主闭环
Beta：第一阶段完整闭环
```

Alpha 先验证报告类数字商品从生产到销端反馈的主链路。Beta 在 Alpha 基础上补齐反馈优化、运营追踪和视频商品包验证。

## 2. Alpha 范围

Alpha 必须实现：

- 供端能力目录、额度样例和默认调用策略。
- `search.web`、`text.generate`、`report.export`、`content.moderate` 四类报告线能力。
- 生产任务最小状态流转。
- 节点执行记录。
- 工具调用和成本记录。
- 质量门禁 `pass`、`manual_review`、`rerun`、`fail`。
- 人工干预暂停与恢复。
- 候选商品包生成、最终质检和正式 `ProductPackage` 固化。
- 深度调研报告生产线 MVP。
- 销端商品列表、详情、预览、模拟购买或免费下载。
- 绑定商品版本的 `MarketFeedback` 回流。

Alpha 可以 mock 或简化：

- provider fallback 可以使用 mock fallback。
- 成本可以使用估算值。
- `content.moderate` 可以使用规则和人工审核混合方式。
- 支付只使用 `simulated_paid` 或 `free_download`。
- 资产服务可以先解析受控 `asset://` 样例。

Alpha 不要求：

- 自动创建优化版本。
- 完整运营仪表盘。
- 视频商品包闭环。
- 真实支付和真实结算。

## 3. Beta 范围

Beta 必须实现：

- 经运营确认后从 `MarketFeedback` 创建优化任务。
- 优化任务记录 `source_feedback_id` 和 `parent_production_task_id`。
- 至少一次优化任务生成补丁版本或新主版本。
- 端到端追踪能回答数据模型中的第一阶段追踪问题。

Beta 可以简化实现：

- 视频线按 `mode_b` 验证。
- 视频生成可使用短片段、受控占位资产或 mock Adapter。
- 端到端追踪可以是后台查询或轻量运营视图，不要求完整仪表盘。

## 4. 已冻结口径

第一阶段已冻结以下口径：

- 报告线主验收路径是深度调研报告生产线。
- 视频线是第二验证路径，不阻塞报告线主闭环。
- 供端最小 provider 范围以报告线四类能力为主。
- `asset://` 表示正式商品资产，`artifact://` 表示产端中间产物。
- 创作者不能执行 `override`，运营人员可以执行 `override`。
- `override` 必须记录 `risk_note`，且不能绕过最终商品质检。
- 第一阶段不接真实支付，使用模拟订单验证权益和下载。
- 反馈触发优化任务必须经过运营确认。

## 5. 开发前必须产出

进入 Alpha 开发前应产出：

- `ToolCapability`、`ResourceQuota`、`ToolInvocationPolicy` 的 v0 JSON 样例。
- 报告线 `WorkflowTemplate` v0。
- `ProductionTask`、`NodeExecution`、`QualityGateResult`、`HumanReviewRecord` 的最小对象定义。
- 报告类 `ProductPackage` 样例。
- 销端 `Listing`、`OrderRecord`、`DownloadRecord`、`MarketFeedback` 的最小对象定义。
- 受控验收数据，包括报告主题、创作者、运营人员、消费者和模拟反馈。

Alpha 开发冻结样例见 [15-alpha-v0-fixtures.md](15-alpha-v0-fixtures.md)。

Alpha 研发任务、依赖和测试矩阵见 [16-alpha-implementation-plan.md](16-alpha-implementation-plan.md)。

Alpha 本地任务发布清单见 [17-alpha-issue-backlog.md](17-alpha-issue-backlog.md)。

Alpha 验收脚本规格见 [18-alpha-acceptance-script-spec.md](18-alpha-acceptance-script-spec.md)。

Alpha 开工前最后确认见 [19-alpha-kickoff-checklist.md](19-alpha-kickoff-checklist.md)。

进入 Beta 开发前应产出：

- 优化任务输入样例。
- 优化版本生成规则样例。
- 端到端追踪查询清单。
- 视频线 `mode_b` 或降级到 `mode_c` 的验收说明。

## 6. 风险与 owner

| 风险 | 影响 | 第一阶段处理 |
| --- | --- | --- |
| provider 不稳定 | 工具节点失败或耗时不可控 | 使用受控 provider 和 mock fallback |
| 资产授权不清 | 商品不能正式上架 | 候选包必须经过最终质检和授权检查 |
| 人工覆盖滥用 | 绕过质量门禁 | `override` 仅限运营，必须记录 `risk_note` |
| 支付范围膨胀 | 拖慢 Alpha | 只做模拟订单和权益记录 |
| 视频成本过高 | 阻塞主闭环 | 视频线不阻塞报告线，允许占位资产 |

## 7. 文档就绪检查

以下检查表示 Alpha 文档口径已经冻结，不代表工程 owner 已完成代码开工 gate。代码开工仍以 [19-alpha-kickoff-checklist.md](19-alpha-kickoff-checklist.md) 的 Code-start Gate 为准。

- [x] v0 契约样例已冻结。
- [x] 报告线 Workflow Template v0 已冻结。
- [x] 供端四类能力的 provider 或 mock provider 已确认。
- [x] 资产 URI 样例和解析方式已确认。
- [x] 人工审核角色和动作权限已确认。
- [x] 模拟订单和下载权益口径已确认。
- [x] Alpha 验收数据已准备。

Beta 开发前检查：

- [ ] Alpha 已通过或有明确豁免项。
- [ ] 优化任务来源字段和版本规则已验证。
- [ ] 运营确认优化任务的入口已确认。
- [ ] 端到端追踪问题已有查询方式。
- [ ] 视频线验收模式已记录。
