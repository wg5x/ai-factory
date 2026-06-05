# Alpha PRD：报告类数字商品主闭环

本文档把现有架构、契约、验收和 Alpha 开工文档收束为一份产品需求文档。它不替代跨端契约、逻辑数据模型、接口边界或验收脚本规格；字段、状态和样例仍以对应专项文档为准。

## 1. 问题陈述

灵枢平台已经完成供端、产端、销端的架构拆分和第一阶段设计，但文档数量较多，团队进入开发前需要一份能统一产品目标、范围、用户价值、验收口径和不做事项的 Alpha PRD。

Alpha 要解决的核心问题是：验证平台能否把供端工具能力组织成产端可恢复、可质检、可人工干预的生产流水线，并最终输出销端可展示、可交付、可反馈回流的标准 `ProductPackage`。

Alpha 不是完整交易市场，也不是完整视频生产平台。Alpha 的主目标是跑通 `research_report` 报告类数字商品主闭环。

## 2. 解决方案

Alpha 实现一条可演示、可验收的报告类数字商品链路：

```text
供端能力可调用
  ↓
产端执行 workflow_research_report_v0
  ↓
人工审核和质量门禁介入
  ↓
生成 ready_for_listing 的 ProductPackage
  ↓
销端上架、预览、模拟购买、下载
  ↓
生成绑定商品版本的 MarketFeedback
```

Alpha 使用冻结样例、受控 provider 或 mock provider、估算成本、受控 `asset://` 和 `artifact://` 解析，以及模拟支付完成端到端验证。

## 3. 目标

- 验证供端可以向产端稳定暴露报告线所需工具能力、额度和调用策略。
- 验证产端可以执行统一 Pipeline Harness，并支持状态追踪、节点记录、人工干预、质量门禁、重跑和商品包固化。
- 验证产端输出的正式 `ProductPackage` 可以被销端独立接入和展示。
- 验证销端可以完成列表、详情、预览、模拟购买、权益、下载和反馈回流。
- 验证 `MarketFeedback` 能绑定到明确的 `product_id`、`package_id` 和 `version`。
- 验证三端边界成立：销端不读取产端内部 `ProductionTask`，不依赖未固化的 `artifact://`。

## 4. 非目标

- 不实现真实支付、退款、结算、打款和财务对账。
- 不实现自动优化任务、`v2` 版本生成或反馈自动重做。
- 不实现视频生产线。
- 不实现完整运营仪表盘、推荐系统、搜索排序、营销活动或创作者店铺。
- 不实现插件市场、外部开放 API 或多团队复杂协作。
- 不把模拟订单作为真实财务结果。

## 5. 用户与角色

| Actor | Role |
| --- | --- |
| 创作者 | 输入报告需求，审核大纲或内容，确认是否继续生产。 |
| 运营人员 | 处理高风险人工审核，必要时执行 `override`，但不能绕过最终质量门禁。 |
| 消费者 | 在销端浏览、预览、模拟购买或下载报告商品，并产生评分评论等反馈。 |
| 供端 owner | 确保工具能力、额度、策略、fallback 和成本记录可用。 |
| 产端 owner | 确保 Workflow Runner、人工干预、质量门禁和商品包固化可用。 |
| 销端 owner | 确保商品接入、展示、模拟订单、权益、下载和反馈回流可用。 |
| QA owner | 确保 S0-S8 验收脚本和证据输出覆盖 Alpha 主链路和边界场景。 |

## 6. 用户故事

1. 作为创作者，我希望提交调研主题和目标受众，以便启动报告生产任务。
2. 作为创作者，我希望平台在必要时澄清报告范围，以便产出符合目标市场。
3. 作为创作者，我希望审核并编辑生成的大纲，以便在正文写作前确认报告结构。
4. 作为创作者，我希望任务在我编辑后继续执行，以便人工干预不导致整条流水线重启。
5. 作为创作者，我希望最终报告商品包包含预览和下载资产，以便它可以成为可售卖的数字商品。
6. 作为运营人员，我希望查看审核点和质量门禁结果，以便在发布前处理风险内容。
7. 作为运营人员，我希望 `override` 必须填写 `risk_note`，以便例外操作可审计。
8. 作为运营人员，我希望 `override` 后仍进入最终质量门禁，以便人工覆盖不能绕过商品安全。
9. 作为供端 owner，我希望暴露 `search.web`、`text.generate`、`report.export` 和 `content.moderate`，以便报告线用最小工具集运行。
10. 作为供端 owner，我希望工具调用记录 provider、fallback、成本、错误和输出引用，以便失败和成本可追踪。
11. 作为产端 owner，我希望每个 workflow 节点都创建 `NodeExecution`，以便任务进度和重跑可审计。
12. 作为产端 owner，我希望质量门禁输出 `pass`、`manual_review`、`rerun` 或 `fail`，以便下游行为确定。
13. 作为产端 owner，我希望重跑创建新的节点执行记录且不覆盖旧产物，以便生产历史可追踪。
14. 作为产端 owner，我希望最终商品包固化为 `ready_for_listing`，以便销端消费稳定的商品对象。
15. 作为销端 owner，我希望只从 `ready_for_listing` 商品包创建 `Listing`，以便未发布或失败商品不能被售卖。
16. 作为销端 owner，我希望商品详情只依赖 `ProductPackage` 和 `Listing`，以便销端不依赖产端内部状态。
17. 作为消费者，我希望在购买或下载前预览报告，以便判断商品是否有用。
18. 作为消费者，我希望模拟购买或免费下载可以生成权益，以便 Alpha 在无真实支付下验证交付。
19. 作为销端 owner，我希望下载记录绑定 `asset_id`、`package_id` 和 `version`，以便交付历史可追踪。
20. 作为产端 owner，我希望 `MarketFeedback` 绑定商品版本，以便后续优化可以正确归因。
21. 作为 QA owner，我希望一个验收入口覆盖 happy path 和异常场景，以便 Alpha 完成状态可客观验证。
22. 作为产品 owner，我希望 Alpha 范围保持在 `research_report`，以便团队完成第一条可演示平台闭环。

## 7. 功能需求

### 7.1 供端能力

- 返回 Alpha 报告线所需 `ToolCapability`、`ResourceQuota` 和 `ToolInvocationPolicy`。
- 支持 `search.web`、`text.generate`、`report.export`、`content.moderate`。
- 支持默认 provider 失败后的 retry 和 fallback 或 mock fallback。
- 工具调用记录必须绑定 `production_task_id` 和 `node_execution_id`。
- 成本可以使用估算值，但必须可追踪。

### 7.2 产端流水线

- 支持创建 `research_report` 生产任务。
- 加载并执行 `workflow_research_report_v0`。
- 支持任务状态 `created`、`running`、`waiting_for_human`、`completed`、`failed`。
- 每个节点执行生成 `NodeExecution`。
- 支持人工审核暂停、恢复和动作权限。
- 支持质量门禁 `pass`、`manual_review`、`rerun`、`fail`。
- 支持按 `target_node_for_rerun` 重跑，且不覆盖旧产物。
- 支持候选商品包、最终质量门禁和正式 `ProductPackage` 固化。

### 7.3 资产和商品包

- `artifact://` 仅表示产端中间产物。
- `asset://` 仅表示正式商品资产。
- 销端只能使用正式 `ProductPackage.assets` 和 `preview_assets` 中的 `asset://`。
- 报告类 `ProductPackage.type_metadata` 至少包含 `page_count`、`format`、`outline_ref`、`citation_summary_ref`。
- 正式商品包状态必须为 `ready_for_listing` 后才能进入销端。

### 7.4 销端接入与交付

- 接收正式 `ProductPackage` 并创建 `Listing`。
- 支持商品列表、详情和预览数据。
- 支持 `simulated_paid` 和 `free_download`。
- 生成 `OrderRecord`、`Entitlement` 和 `DownloadRecord`。
- 订单、权益、下载必须绑定同一 `package_id` 和 `version`。

### 7.5 反馈回流

- 销端聚合浏览、购买、下载、评分、评论等数据。
- 生成并回传 `MarketFeedback`。
- `MarketFeedback` 必须绑定 `product_id`、`package_id` 和 `version`。
- Alpha 中 feedback 不直接创建优化任务，不修改 `WorkflowTemplate`。

## 8. Alpha 验收标准

Alpha 完成必须同时满足：

- [ ] A0-A12 研发任务全部完成。
- [ ] S0-S8 验收脚本全部通过。
- [ ] 报告主闭环可从输入主题跑到 `MarketFeedback` 回流。
- [ ] `ProductionTask.status = completed`。
- [ ] `ProductPackage.status = ready_for_listing`。
- [ ] `Listing.listing_status = listed`。
- [ ] 订单、权益、下载和反馈绑定同一个 `package_id` 和 `version`。
- [ ] 工具调用成本绑定 `production_task_id` 和 `node_execution_id`。
- [ ] 人工干预不能绕过最终质量门禁。
- [ ] 销端不读取 `ProductionTask` 或未固化 `artifact://`。
- [ ] 已上架商品版本不会被原地覆盖。

## 9. 实施决策

- Alpha 只实现 `research_report` 产品线。
- Alpha 固定使用 `workflow_research_report_v0`。
- Alpha 使用冻结 ID 和样例：`task_research_ai_km_001`、`product_research_ai_km_001`、`package_research_ai_km_001_v1`、`listing_research_ai_km_001_v1`、`feedback_research_ai_km_001_2026_06_05`。
- 供端最小能力为 `search.web`、`text.generate`、`report.export`、`content.moderate`。
- `search.web` 和 `text.generate` 至少支持 fallback 或 mock fallback。
- `report.export` 可使用平台导出 Adapter。
- `content.moderate` 可使用规则和人工审核混合实现。
- 创作者可以执行 `approve`、`edit`、`choose`、`rerun`。
- 运营人员可以执行 `approve`、`edit`、`choose`、`rerun`、`reject`、`override`。
- `override` 必须记录 `risk_note`，且不能绕过 `final_quality_gate`。
- 支付只支持 `simulated_paid` 和 `free_download`。
- `MarketFeedback` 不自动创建优化任务；优化任务进入 Beta。

## 10. 测试决策

- 测试只验证外部行为和跨端契约，不绑定内部实现细节。
- S0 校验冻结样例和 ID 闭合。
- S1 覆盖报告主闭环。
- S2 覆盖质量门禁重跑。
- S3 覆盖人工编辑恢复。
- S4 覆盖运营覆盖边界。
- S5 覆盖工具 fallback 和成本记录。
- S6 覆盖模拟购买、权益和下载。
- S7 覆盖反馈回流和版本绑定。
- S8 覆盖销端边界守卫。
- 验收失败时必须能定位到 A0-A12 的具体任务。

## 11. 风险

| Risk | Impact | Alpha Handling |
| --- | --- | --- |
| provider 不稳定 | 工具节点失败或耗时不可控 | 使用受控 provider 和 mock fallback |
| 资产授权不清 | 商品不能进入正式交付 | 最终质量门禁检查授权和资产状态 |
| 人工覆盖滥用 | 绕过质量控制 | 限制 `override` 权限并要求 `risk_note` |
| 支付范围膨胀 | 拖慢 Alpha | 只做模拟订单和权益记录 |
| 反馈无法归因 | 后续优化无法追踪 | feedback 必须绑定商品版本 |

## 12. 参考文档

- [CONTEXT.md](CONTEXT.md)
- [05-contracts.md](05-contracts.md)
- [06-pipeline-runtime.md](06-pipeline-runtime.md)
- [10-interface-boundaries.md](10-interface-boundaries.md)
- [12-first-phase-acceptance-plan.md](12-first-phase-acceptance-plan.md)
- [14-development-readiness.md](14-development-readiness.md)
- [15-alpha-v0-fixtures.md](15-alpha-v0-fixtures.md)
- [16-alpha-implementation-plan.md](16-alpha-implementation-plan.md)
- [17-alpha-issue-backlog.md](17-alpha-issue-backlog.md)
- [18-alpha-acceptance-script-spec.md](18-alpha-acceptance-script-spec.md)
- [19-alpha-kickoff-checklist.md](19-alpha-kickoff-checklist.md)
