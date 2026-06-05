# Alpha TODO List

本文档把 Alpha PRD 和既有实施计划收束为一份可执行待办清单。它面向开工、排期和验收使用；任务细节仍以 [16-alpha-implementation-plan.md](16-alpha-implementation-plan.md)、[17-alpha-issue-backlog.md](17-alpha-issue-backlog.md) 和 [18-alpha-acceptance-script-spec.md](18-alpha-acceptance-script-spec.md) 为准。

## 1. 使用方式

- 从上到下推进，先完成 Code-start Gate，再进入 A0-A12。
- 每个任务完成时同时勾选实现项和验收项。
- 如果任务范围扩大，先回到 [PRD.md](PRD.md) 和 [14-development-readiness.md](14-development-readiness.md) 确认。
- 如果样例、ID 或验收脚本变化，先更新对应冻结文档，不在实现中临时漂移。

## 2. Code-start Gate

- [ ] 产品 owner 确认 Alpha 只做 `research_report` 主闭环。
- [ ] 研发 owner 确认 A0-A12 owner 已分配。
- [ ] 供端 owner 确认四类能力可用或可 mock。
- [ ] 产端 owner 确认 `workflow_research_report_v0` 节点结构冻结。
- [ ] 销端 owner 确认只依赖正式 `ProductPackage`。
- [ ] QA owner 确认 S0-S8 验收脚本规格。
- [ ] 运营审核 owner 确认 `override` 权限和 `risk_note` 规则。
- [ ] `15-alpha-v0-fixtures.md` 中全部 JSON 样例可解析。
- [ ] Alpha 主链路 ID 闭合。
- [ ] `asset://` 和 `artifact://` 解析规则已确认。

## 3. Milestones

| Milestone | Goal | Exit Criteria |
| --- | --- | --- |
| M0 样例和契约校验 | 开发输入稳定 | A0 完成，固定 ID 闭合 |
| M1 供端和产端骨架 | 报告任务可执行到人工暂停 | A1、A3、A4、A5 完成 |
| M2 质量和人工闭环 | 流水线可恢复、可重跑、可审计 | A2、A6、A7 完成 |
| M3 商品包和销端接入 | 正式商品可上架展示 | A8、A9 完成 |
| M4 交付和反馈回流 | 主闭环跑到 feedback | A10、A11、A12 完成 |

## 4. Task Checklist

### A0 冻结样例校验器

Blocked by：None

验收脚本：S0 fixture_validation

- [ ] 实现读取 `15-alpha-v0-fixtures.md` 的校验入口。
- [ ] 提取并解析全部 `json` 代码块。
- [ ] 校验 `product_research_ai_km_001`、`package_research_ai_km_001_v1`、`task_research_ai_km_001` 闭合。
- [ ] 校验 `workflow_template_id`、`product_id`、`package_id`、`production_task_id` 关联一致。
- [ ] 校验失败时输出失败 JSON block 或缺失 ID。
- [ ] S0 输出 `json_blocks_total`、`product_id`、`package_id`、`production_task_id` 和校验状态。

### A1 供端能力目录

Blocked by：A0

验收脚本：S1 report_happy_path

- [ ] 支持按 `research_report` 查询工具能力。
- [ ] 返回 `search.web`、`text.generate`、`report.export`、`content.moderate`。
- [ ] 返回对应 `ToolCapability`、`ResourceQuota`、`ToolInvocationPolicy`。
- [ ] 限制 `research_report` 只能拿到 Alpha 允许能力。
- [ ] 查询额度时能区分可用、已用和预留额度。

### A2 供端工具调用与 fallback

Blocked by：A1

验收脚本：S5 tool_fallback

- [ ] 支持工具节点调用供端能力。
- [ ] 记录 provider、错误、成本和最终 `output_ref`。
- [ ] 工具调用记录绑定 `production_task_id`。
- [ ] 工具调用记录绑定 `node_execution_id`。
- [ ] 默认 provider 超时后按策略重试。
- [ ] 重试失败后进入 fallback provider 或 mock fallback。
- [ ] 产端只消费最终 `output_ref`。

### A3 资产引用服务

Blocked by：A0

验收脚本：S1 report_happy_path、S8 boundary_guard

- [ ] 支持 Alpha 受控 `artifact://` 解析。
- [ ] 支持 Alpha 受控 `asset://` 解析。
- [ ] 区分 `intermediate`、`candidate_package`、`official_package`。
- [ ] 限制 `artifact://` 只供产端和运营审核视图使用。
- [ ] 确认销端预览和下载只能使用正式商品包里的 `asset://`。

### A4 生产任务运行骨架

Blocked by：A0

验收脚本：S1 report_happy_path

- [ ] 支持创建 `research_report` 生产任务。
- [ ] 支持 `created`、`running`、`waiting_for_human`、`completed`、`failed`。
- [ ] 每个节点执行生成 `NodeExecution`。
- [ ] 保存当前节点、失败原因和恢复入口。
- [ ] 查询生产任务时能返回状态、当前节点和最新商品包引用。

### A5 报告线 Workflow Runner

Blocked by：A1、A4

验收脚本：S1 report_happy_path、S3 human_edit_resume

- [ ] 加载 `workflow_research_report_v0`。
- [ ] 按冻结模板执行 `input_intake` 到 `outline_review`。
- [ ] 到 `outline_review` 时进入 `waiting_for_human`。
- [ ] 查询当前节点和等待原因。
- [ ] 人工节点不被当作流程外补丁处理。

### A6 质量门禁与重跑

Blocked by：A5

验收脚本：S2 quality_gate_rerun

- [ ] 支持门禁结果 `pass`。
- [ ] 支持门禁结果 `manual_review`。
- [ ] 支持门禁结果 `rerun`。
- [ ] 支持门禁结果 `fail`。
- [ ] `fact_check` 可输出 `rerun`。
- [ ] `rerun` 记录 `reason` 和 `target_node_for_rerun`。
- [ ] 重跑目标节点及后续自动节点。
- [ ] 被重跑节点生成新的 `NodeExecution`，不覆盖旧产物。

### A7 人工干预恢复

Blocked by：A5、A6

验收脚本：S3 human_edit_resume、S4 operator_override_guard

- [ ] creator 可执行 `approve`、`edit`、`choose`、`rerun`。
- [ ] operator 可执行 `approve`、`edit`、`choose`、`rerun`、`reject`、`override`。
- [ ] `reject` 必须记录原因。
- [ ] `edit` 生成新的 `output_ref`。
- [ ] `override` 必须记录 `risk_note`。
- [ ] creator 不能执行 `override`。
- [ ] `override` 后仍进入 `final_quality_gate`。
- [ ] `HumanReviewRecord` 记录操作者、动作、原因和恢复节点。

### A8 商品包生成与固化

Blocked by：A3、A6、A7

验收脚本：S1 report_happy_path、S4 operator_override_guard、S8 boundary_guard

- [ ] 生成候选商品包时资产范围为 `candidate_package`。
- [ ] 候选包进入最终质量门禁。
- [ ] 最终质量门禁通过后固化正式 `ProductPackage`。
- [ ] 正式商品包状态为 `ready_for_listing`。
- [ ] 报告类 `type_metadata` 包含页数、格式、大纲引用和引用摘要引用。
- [ ] 正式资产使用 `asset://`。
- [ ] 生产任务完成后进入 `completed`。
- [ ] 已上架商品版本不会被原地覆盖。

### A9 销端商品接入与展示

Blocked by：A8

验收脚本：S1 report_happy_path、S6 purchase_download、S8 boundary_guard

- [ ] 只接收 `ready_for_listing` 商品包。
- [ ] 创建 `listing_research_ai_km_001_v1`。
- [ ] Listing 状态为 `listed`。
- [ ] 商品列表展示标题、封面、简介、标签和价格信息。
- [ ] 商品详情展示预览资产、页数、格式和下载入口。
- [ ] 商品详情只依赖 `ProductPackage` 和 `Listing`。
- [ ] 销端不读取 `ProductionTask`。

### A10 模拟购买、权益和下载

Blocked by：A9

验收脚本：S6 purchase_download

- [ ] 支持 `simulated_paid` 模拟购买。
- [ ] 支持 `free_download` 免费下载。
- [ ] 生成 `OrderRecord`。
- [ ] 生成与订单绑定的 `Entitlement`。
- [ ] 生成绑定 `asset_report_ai_km_pdf_001` 的 `DownloadRecord`。
- [ ] 订单、权益、下载记录绑定同一 `package_id` 和 `version`。
- [ ] 模拟支付不进入真实财务结算。

### A11 MarketFeedback 回流

Blocked by：A10

验收脚本：S7 feedback_return

- [ ] 聚合浏览、购买、下载、评分和评论数据。
- [ ] 生成 `feedback_research_ai_km_001_2026_06_05`。
- [ ] feedback 绑定 `product_id`。
- [ ] feedback 绑定 `package_id`。
- [ ] feedback 绑定 `version`。
- [ ] feedback 回传产端后不直接创建优化任务。
- [ ] feedback 不修改 `WorkflowTemplate`。

### A12 Alpha 验收脚本

Blocked by：A1-A11

验收脚本：S0-S8

- [ ] 实现或配置 S0 `fixture_validation`。
- [ ] 实现或配置 S1 `report_happy_path`。
- [ ] 实现或配置 S2 `quality_gate_rerun`。
- [ ] 实现或配置 S3 `human_edit_resume`。
- [ ] 实现或配置 S4 `operator_override_guard`。
- [ ] 实现或配置 S5 `tool_fallback`。
- [ ] 实现或配置 S6 `purchase_download`。
- [ ] 实现或配置 S7 `feedback_return`。
- [ ] 实现或配置 S8 `boundary_guard`。
- [ ] 每个脚本输出可追踪证据。
- [ ] 失败时能定位到 A0-A12 的具体任务。
- [ ] 脚本不依赖真实支付。
- [ ] 脚本不依赖视频线。

## 5. Recommended Order

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

A0 完成后可以并行启动 A1、A3、A4。A12 只在 A1-A11 完成后执行。

## 6. Done Definition

- [ ] A0-A12 全部完成。
- [ ] S0-S8 全部通过。
- [ ] 报告主闭环连续跑通至少 1 次。
- [ ] T2、T3、T4、T5、T6、T7 场景全部通过。
- [ ] 每个验收脚本都有证据输出。
- [ ] Alpha 不包含任何明确列入不做清单的能力。
