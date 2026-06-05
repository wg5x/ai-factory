# Alpha 开工清单

本文档用于开始写代码前的最后一次确认。

通过本文档检查后，才进入 Alpha 代码实现；未通过项应回到对应设计文档补齐，而不是在开发中临时漂移。

## 1. 开工范围

Alpha 只做报告类数字商品主闭环：

```text
research_report
  -> workflow_research_report_v0
  -> package_research_ai_km_001_v1
  -> listing_research_ai_km_001_v1
  -> feedback_research_ai_km_001_2026_06_05
```

Alpha 开工必须以以下文档为准：

- [14-development-readiness.md](14-development-readiness.md)
- [15-alpha-v0-fixtures.md](15-alpha-v0-fixtures.md)
- [16-alpha-implementation-plan.md](16-alpha-implementation-plan.md)
- [17-alpha-issue-backlog.md](17-alpha-issue-backlog.md)
- [18-alpha-acceptance-script-spec.md](18-alpha-acceptance-script-spec.md)

## 2. Owner 确认

| 角色 | 开工前职责 | 最小确认 |
| --- | --- | --- |
| 产品 owner | 确认 Alpha 范围、用户链路和不做清单 | `research_report` 主闭环范围不再扩大 |
| 研发 owner | 确认 A0-A12 拆分、依赖和并行策略 | A0、A1、A3、A4 可先启动 |
| 供端 owner | 确认四类能力和 provider 口径 | `search.web`、`text.generate`、`report.export`、`content.moderate` 可用或可 mock |
| 产端 owner | 确认 Workflow Runner、人工干预和质量门禁口径 | `workflow_research_report_v0` 不再改节点结构 |
| 销端 owner | 确认 listing、模拟订单、权益、下载和反馈口径 | 销端只依赖正式 `ProductPackage` |
| QA owner | 确认 S0-S8 验收脚本规格 | 每个脚本都有步骤、断言和证据输出 |
| 运营审核 owner | 确认人工审核角色和 `override` 规则 | `override` 必须有 `risk_note`，不能绕过最终质检 |

## 3. 已冻结口径

开工后默认冻结：

- Alpha 只实现 `research_report`。
- Alpha 只使用 `workflow_research_report_v0`。
- Alpha 使用 [15-alpha-v0-fixtures.md](15-alpha-v0-fixtures.md) 中固定 ID。
- `asset://` 只表示正式商品资产。
- `artifact://` 只表示产端中间产物。
- 创作者不能执行 `override`。
- 运营人员可以执行 `override`，但必须记录 `risk_note`。
- `override` 不能绕过 `final_quality_gate`。
- 支付只支持 `simulated_paid` 和 `free_download`。
- `MarketFeedback` 不自动创建优化任务。
- 视频线不进入 Alpha。

## 4. 开发环境准备

开工前需要准备：

- 本地文档以当前编号 00-19 为准。
- A0 先实现样例校验入口，再开始依赖样例的业务实现。
- 供端 mock provider 或受控 provider 已能覆盖四类能力。
- `asset://` 和 `artifact://` 至少有受控解析规则。
- 测试环境可以创建生产任务、商品包、listing、订单、权益、下载记录和反馈记录。
- 验收脚本可以输出结构化证据，便于定位 A0-A12 中的失败任务。

## 5. Issue 发布检查

发布正式 issue 或 sprint item 前检查：

- [ ] A0-A12 已按 [17-alpha-issue-backlog.md](17-alpha-issue-backlog.md) 拆成独立任务。
- [ ] 每个任务都有 `Blocked by`、`What to build` 和 `Acceptance criteria`。
- [ ] A0 排在第一位。
- [ ] A1、A3、A4 可在 A0 通过后并行。
- [ ] A12 只在 A1-A11 完成后执行。
- [ ] 每个 issue 都能追踪到 S0-S8 中至少一个验收脚本或测试矩阵场景。

## 6. Code-start Gate

满足以下条件后，可以开始写代码：

- [ ] `15-alpha-v0-fixtures.md` 中全部 JSON 样例可解析。
- [ ] Alpha 主链路 ID 闭合。
- [ ] A0-A12 的 owner 已分配。
- [ ] S0-S8 的验收脚本规格已确认。
- [ ] 供端 mock 或受控 provider 已确认。
- [ ] 资产 URI 解析规则已确认。
- [ ] 人工干预权限已确认。
- [ ] 销端不读取 `ProductionTask` 和未固化 `artifact://` 的边界已确认。
- [ ] 产品 owner 确认 Alpha 不新增范围。
- [ ] 研发 owner 确认从 A0 开始实现。

## 7. 暂不开发

Alpha 开工时明确不开发：

- 真实支付、退款、结算和打款。
- 自动优化任务和 `v2` 版本生成。
- 视频生产线。
- 完整运营仪表盘。
- 创作者店铺。
- 推荐、搜索排序和营销活动。
- 外部开放 API。
- 多团队复杂协作。

## 8. 开工后变更规则

开工后如果需要变更：

- 范围变化，先改 [14-development-readiness.md](14-development-readiness.md)。
- 样例或 ID 变化，先改 [15-alpha-v0-fixtures.md](15-alpha-v0-fixtures.md)。
- 任务依赖变化，先改 [16-alpha-implementation-plan.md](16-alpha-implementation-plan.md) 和 [17-alpha-issue-backlog.md](17-alpha-issue-backlog.md)。
- 验收脚本变化，先改 [18-alpha-acceptance-script-spec.md](18-alpha-acceptance-script-spec.md)。
- 跨端契约变化，先回到 [05-contracts.md](05-contracts.md)、[09-logical-data-model.md](09-logical-data-model.md) 和 [10-interface-boundaries.md](10-interface-boundaries.md) 评估影响。

任何会扩大 Alpha 范围的变更，都应先由产品 owner 和研发 owner 共同确认。
