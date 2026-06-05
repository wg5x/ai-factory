# Alpha 实施计划

本文档把 Alpha 从冻结样例推进到可排期、可验收的研发任务。

Alpha 的目标是跑通报告类数字商品主闭环：

```text
供端能力可调用
  ↓
产端执行报告线 Workflow
  ↓
生成正式 ProductPackage
  ↓
销端上架、预览、模拟购买、下载
  ↓
生成绑定版本的 MarketFeedback
```

## 1. 实施假设

- Alpha 只实现 `research_report` 产品线。
- Alpha 使用 [15-alpha-v0-fixtures.md](15-alpha-v0-fixtures.md) 中冻结的 ID、JSON 样例和验收数据。
- Alpha 任务发布可参考 [17-alpha-issue-backlog.md](17-alpha-issue-backlog.md)。
- Alpha 验收脚本规格见 [18-alpha-acceptance-script-spec.md](18-alpha-acceptance-script-spec.md)。
- 供端 provider 可以使用受控 provider 或 mock provider。
- 成本记录可以使用估算值。
- 资产服务可以先支持受控 `asset://` 和 `artifact://` 解析。
- 支付只支持 `simulated_paid` 和 `free_download`。
- Alpha 不实现优化任务、完整运营仪表盘、视频线、真实支付和真实结算。

## 2. 开发顺序

建议按四个检查点推进：

1. 样例和契约校验。
2. 供端能力和产端流水线。
3. 商品包生成和销端接入。
4. 购买下载和反馈回流。

每个检查点都应能独立演示，不等到最后才联调。

## 3. 任务拆解

| 任务 | 内容 | 依赖 | 验收 |
| --- | --- | --- | --- |
| A0 样例校验器 | 解析 `15-alpha-v0-fixtures.md` 中所有 JSON 样例，校验关键 ID 闭合 | 无 | 17 个 JSON 样例可解析；`product_id`、`package_id`、`production_task_id` 闭合 |
| A1 供端能力目录 | 返回 `search.web`、`text.generate`、`report.export`、`content.moderate` 能力和策略 | A0 | 能按 `research_report` 查询能力、额度和策略 |
| A2 供端工具调用 | 支持工具调用记录、成本估算、失败重试和 fallback | A1 | 默认 provider 超时后能走 fallback，产端只拿最终 `output_ref` |
| A3 资产引用服务 | 支持 `artifact://` 中间产物和 `asset://` 商品资产的受控解析 | A0 | 销端只能读取正式 `ProductPackage` 中的 `asset://` |
| A4 生产任务骨架 | 支持 `ProductionTask` 创建、状态流转和 `NodeExecution` 记录 | A0 | 任务能进入 `running`、`waiting_for_human`、`completed`、`failed` |
| A5 Workflow Runner | 加载 `workflow_research_report_v0` 并按节点顺序执行 | A1、A4 | 能执行到 `outline_review` 并暂停 |
| A6 质量门禁 | 支持 `pass`、`manual_review`、`rerun`、`fail` 和重跑目标 | A5 | `fact_check` 可输出 `rerun` 并生成新 `NodeExecution` |
| A7 人工干预 | 支持 `approve`、`edit`、`rerun`、`reject`、运营 `override` | A5、A6 | `edit` 生成新 `output_ref`；`override` 记录 `risk_note` 且不绕过最终质检 |
| A8 商品包生成 | 支持候选包、最终质量门禁、正式 `ProductPackage` 固化 | A3、A6、A7 | 生成 `ready_for_listing` 的 `package_research_ai_km_001_v1` |
| A9 销端接入 | 接收正式 `ProductPackage` 并创建 `Listing` | A8 | listing 状态为 `listed`，不读取产端任务状态 |
| A10 购买下载 | 支持模拟订单、权益和下载记录 | A9 | `OrderRecord`、`Entitlement`、`DownloadRecord` 绑定同一 `package_id` 和 `version` |
| A11 反馈回流 | 聚合并回传 `MarketFeedback` | A10 | feedback 绑定 `product_id`、`package_id`、`version` |
| A12 Alpha 验收脚本 | 串起报告生产闭环、异常场景和销端反馈 | A1-A11 | 按 `18-alpha-acceptance-script-spec.md` 覆盖 S0-S8 |

## 4. 并行建议

可以并行推进：

- 供端线：A1、A2。
- 产端线：A4、A5、A6、A7。
- 资产和销端线：A3、A9、A10、A11。

必须串行的关键路径：

```text
A0
  ↓
A1 + A4
  ↓
A5
  ↓
A6 + A7
  ↓
A8
  ↓
A9
  ↓
A10
  ↓
A11
  ↓
A12
```

## 5. 最小接口面

Alpha 至少需要以下接口或等价调用方式：

| 端 | 能力 | 说明 |
| --- | --- | --- |
| 供端 | 查询能力 | 返回 `ToolCapability[]` |
| 供端 | 查询额度 | 返回 `ResourceQuota[]` |
| 供端 | 解析策略 | 返回 `ToolInvocationPolicy` |
| 供端 | 调用工具 | 返回 `output_ref`、成本和错误 |
| 产端 | 创建生产任务 | 使用 `research_report` 和 `workflow_research_report_v0` |
| 产端 | 查询生产任务 | 返回状态、当前节点、最新商品包 |
| 产端 | 提交人工干预 | 支持 creator 和 operator 动作权限 |
| 产端 | 查询商品包 | 只返回正式 `ProductPackage` |
| 销端 | 接入商品包 | 只接收 `ready_for_listing` |
| 销端 | 查询列表和详情 | 只依赖 `ProductPackage` 和 `Listing` |
| 销端 | 记录购买下载 | 生成订单、权益和下载记录 |
| 销端 | 回传反馈 | 生成 `MarketFeedback` |

## 6. 测试矩阵

| 场景 | 输入 | 通过标准 |
| --- | --- | --- |
| T1 报告主闭环 | `AI Agent 在企业知识管理中的应用` | 任务 `completed`，商品包 `ready_for_listing`，listing `listed`，feedback 绑定版本 |
| T2 质量门禁重跑 | `fact_check` 来源不足 | 记录 `target_node_for_rerun`，重跑节点生成新 `NodeExecution`，不覆盖旧产物 |
| T3 人工编辑恢复 | `outline_review` 进入等待 | `edit` 生成新 `output_ref`，从 `report_writing` 恢复 |
| T4 运营覆盖 | `final_review` 由 operator 执行 `override` | 记录 `risk_note`，后续仍进入 `final_quality_gate` |
| T5 工具 fallback | 默认 `search.web` provider 超时 | 策略重试后走 fallback，成本绑定节点执行 |
| T6 模拟购买下载 | user_001 购买并下载报告 | 订单、权益、下载记录绑定同一 `package_id` 和 `version` |
| T7 反馈回流 | 销端生成反馈窗口 | `MarketFeedback` 绑定 `product_id`、`package_id`、`version` |

## 7. 不做清单

Alpha 明确不做：

- 自动从 `MarketFeedback` 创建优化任务。
- 生成 `v2` 或补丁版本。
- 视频生成生产线。
- 真实支付、退款、结算和打款。
- 创作者店铺、推荐、搜索排序和运营仪表盘。
- 多团队复杂协作和外部开放 API。

## 8. 开发完成标准

Alpha 视为完成，需要同时满足：

- [ ] A0-A12 全部完成。
- [ ] `18-alpha-acceptance-script-spec.md` 中 S0-S8 全部通过。
- [ ] 第 6 节测试矩阵全部通过。
- [ ] `15-alpha-v0-fixtures.md` 中的 Alpha 链路 ID 在实际记录中保持一致。
- [ ] 销端没有读取 `ProductionTask` 或中间 `artifact://` 作为展示依赖。
- [ ] 人工干预不能绕过最终质量门禁。
- [ ] 工具调用成本能追踪到 `production_task_id` 和 `node_execution_id`。
- [ ] 已上架商品版本不会被原地覆盖。

## 9. 后续转 Beta 条件

进入 Beta 前，Alpha 至少需要：

- 报告主闭环连续跑通 3 次。
- T2、T3、T4 三个异常场景至少各通过 1 次。
- `MarketFeedback` 能稳定归因到商品版本。
- 运营确认优化任务的入口和字段已准备好。
