# Alpha 验收脚本规格

本文档定义 Alpha 进入开发后需要实现的验收脚本规格。

它不是代码，不规定测试框架，只说明脚本入口、输入、步骤、断言和证据输出。

## 1. 输入基准

所有验收脚本使用以下文档作为输入基准：

- [15-alpha-v0-fixtures.md](15-alpha-v0-fixtures.md)
- [16-alpha-implementation-plan.md](16-alpha-implementation-plan.md)
- [17-alpha-issue-backlog.md](17-alpha-issue-backlog.md)

固定验收 ID：

```text
creator_id：creator_001
operator_id：operator_001
user_id：user_001
product_line_id：research_report
workflow_template_id：workflow_research_report_v0
production_task_id：task_research_ai_km_001
product_id：product_research_ai_km_001
package_id：package_research_ai_km_001_v1
listing_id：listing_research_ai_km_001_v1
feedback_id：feedback_research_ai_km_001_2026_06_05
```

## 2. 脚本清单

| 脚本 | 覆盖任务 | 目的 |
| --- | --- | --- |
| S0 fixture_validation | A0 | 校验冻结样例可解析且 ID 闭合 |
| S1 report_happy_path | A1-A11 | 跑通报告主闭环 |
| S2 quality_gate_rerun | A6 | 验证质量门禁重跑 |
| S3 human_edit_resume | A7 | 验证人工编辑恢复 |
| S4 operator_override_guard | A7、A8 | 验证运营覆盖不能绕过最终质检 |
| S5 tool_fallback | A2 | 验证工具 fallback 和成本记录 |
| S6 purchase_download | A9、A10 | 验证模拟购买、权益和下载 |
| S7 feedback_return | A10、A11 | 验证反馈回流和版本绑定 |
| S8 boundary_guard | A8-A11 | 验证销端不读取产端中间状态 |

## 3. S0 fixture_validation

步骤：

1. 读取 [15-alpha-v0-fixtures.md](15-alpha-v0-fixtures.md)。
2. 提取全部 `json` 代码块。
3. 逐个解析 JSON。
4. 校验 `product_id`、`package_id`、`production_task_id`、`workflow_template_id` 是否闭合。

断言：

- 所有 JSON 块可解析。
- Alpha 链路只使用 `product_research_ai_km_001` 和 `package_research_ai_km_001_v1`。
- 所有产端运行对象都能追踪到 `task_research_ai_km_001`。

证据输出：

```text
json_blocks_total
product_id
package_id
production_task_id
fixture_validation_status
```

## 4. S1 report_happy_path

步骤：

1. 创建 `research_report` 生产任务。
2. 查询供端能力、额度和策略。
3. 执行 `workflow_research_report_v0`。
4. 经过至少一个人工审核点。
5. 生成候选包。
6. 通过最终质量门禁。
7. 固化正式 `ProductPackage`。
8. 销端接入商品包并创建 `Listing`。
9. 生成模拟订单、权益和下载记录。
10. 生成并回传 `MarketFeedback`。

断言：

- `ProductionTask.status = completed`。
- `ProductPackage.status = ready_for_listing`。
- `Listing.listing_status = listed`。
- 订单、权益、下载和反馈都绑定同一个 `package_id` 和 `version`。
- 销端展示不依赖 `ProductionTask`。

证据输出：

```text
production_task_id
package_id
listing_id
order_id
entitlement_id
download_id
feedback_id
```

## 5. S2 quality_gate_rerun

步骤：

1. 构造 `fact_check` 来源不足场景。
2. 让质量门禁输出 `rerun`。
3. 指定 `target_node_for_rerun = source_search`。
4. 重跑目标节点及后续自动节点。

断言：

- `QualityGateResult.decision = rerun`。
- `target_node_for_rerun` 不为空。
- 被重跑节点生成新的 `NodeExecution`。
- 旧的成功产物没有被覆盖。

## 6. S3 human_edit_resume

步骤：

1. 让 `outline_review` 进入 `waiting_for_human`。
2. 使用 `creator_001` 提交 `edit`。
3. 生成新的 `output_ref`。
4. 从 `report_writing` 恢复。

断言：

- `HumanReviewRecord.action = edit`。
- `HumanReviewRecord.operator_role = creator`。
- `output_ref` 指向人工修改后的新产物。
- 恢复节点为 `report_writing`。

## 7. S4 operator_override_guard

步骤：

1. 让 `final_review` 进入人工审核。
2. 使用 `operator_001` 提交 `override` 和 `risk_note`。
3. 继续到 `package_report_candidate`。
4. 再进入 `final_quality_gate`。

断言：

- `creator` 不能执行 `override`。
- `operator` 执行 `override` 时必须提供 `risk_note`。
- `override` 不能跳过 `final_quality_gate`。
- 最终质检不通过时不能固化正式商品包。

## 8. S5 tool_fallback

步骤：

1. 模拟默认 `search.web` provider 超时。
2. 按 `ToolInvocationPolicy.retry_policy` 重试。
3. 重试失败后调用 fallback provider。

断言：

- 工具调用记录包含默认 provider 和 fallback provider。
- 成本记录绑定 `production_task_id` 和 `node_execution_id`。
- 产端只使用最终 `output_ref`。

## 9. S6 purchase_download

步骤：

1. 销端使用 `package_research_ai_km_001_v1` 创建 `Listing`。
2. `user_001` 创建模拟订单。
3. 生成权益记录。
4. 下载 `asset_report_ai_km_pdf_001`。

断言：

- `OrderRecord.payment_status = simulated_paid`。
- `Entitlement.source_order_id` 指向模拟订单。
- `DownloadRecord.asset_id = asset_report_ai_km_pdf_001`。
- 订单、权益和下载绑定同一商品版本。

## 10. S7 feedback_return

步骤：

1. 聚合浏览、购买、下载、评分和评论数据。
2. 生成 `feedback_research_ai_km_001_2026_06_05`。
3. 回传给产端。

断言：

- `MarketFeedback` 绑定 `product_id`、`package_id` 和 `version`。
- feedback 不直接创建优化任务。
- feedback 不修改 `WorkflowTemplate`。

## 11. S8 boundary_guard

步骤：

1. 查询销端商品列表。
2. 查询销端商品详情。
3. 查询下载入口。

断言：

- 销端只使用 `ProductPackage`、`Listing` 和自有记录。
- 销端不读取 `ProductionTask`。
- 销端不读取未进入正式商品包的 `artifact://`。

## 12. 通过标准

Alpha 验收脚本通过需要满足：

- [ ] S0-S8 全部通过。
- [ ] 每个脚本输出可追踪证据。
- [ ] 失败时能定位到任务编号 A0-A12。
- [ ] 脚本不依赖真实支付。
- [ ] 脚本不依赖视频线。
