# 待决事项与评审清单

本文档集中记录第一阶段进入开发前需要人工确认的决策、风险和评审清单。

当某个问题被确认后，应同步更新对应设计文档，而不是只修改本文档。

## 1. 决策状态

```text
open：尚未确认。
proposed：已有建议，等待确认。
decided：已确认，需同步到对应文档。
deferred：明确后置，不阻塞第一阶段。
```

## 2. 必须确认的决策

### 2.1 第一阶段主演示产品线

状态：decided

决策：

- 深度调研报告生产线作为主演示产品线。
- 视频生成生产线作为第二验证路径，可以使用受控占位资产或短片段。
- 视频线不阻塞报告线主闭环验收。

影响：

- 第一阶段资源优先投向报告线。
- 验收计划中视频线按可简化验证处理，不要求完整真实生成。

已同步：

- [07-first-phase-slices.md](07-first-phase-slices.md)
- [11-product-line-mvp-specs.md](11-product-line-mvp-specs.md)
- [12-first-phase-acceptance-plan.md](12-first-phase-acceptance-plan.md)

### 2.2 视频生成生产线验收模式

状态：decided

可选方案：

```text
mode_a：真实生成完整视频。
mode_b：真实生成脚本、分镜和部分片段，最终视频用受控占位资产合成。
mode_c：只跑通视频商品包闭环，不调用高成本视频生成。
```

决策：

- 第一阶段采用 `mode_b`。
- 如果 provider、成本或素材授权暂不满足，可以降级到 `mode_c`，但必须在验收记录中标明。

影响：

- 影响成本、工具 provider 选择、验收时间和质量门禁严格程度。

已同步：

- [11-product-line-mvp-specs.md](11-product-line-mvp-specs.md)
- [12-first-phase-acceptance-plan.md](12-first-phase-acceptance-plan.md)

### 2.3 第一阶段供端 provider 范围

状态：open

需要确认：

- `search.web` 使用哪些 provider。
- `text.generate` 使用哪些 provider。
- `report.export` 是否自研，还是复用第三方导出服务。
- 视频相关能力是否接入真实 provider。

建议：

- 报告线先只接入 `search.web`、`text.generate`、`report.export`、`content.moderate` 的最小 provider。
- 视频线 provider 视 `mode_b` 或 `mode_c` 决定。

影响：

- 影响 `ToolInvocationPolicy` 样例和 fallback 策略。
- 影响工具调用成本追踪。

需要更新：

- [02-supply-platform.md](02-supply-platform.md)
- [08-contract-examples.md](08-contract-examples.md)
- [10-interface-boundaries.md](10-interface-boundaries.md)

### 2.4 资产存储和 URI 规范

状态：proposed

需要确认：

- `asset://` 是否作为内部统一 URI scheme。
- 资产是否区分中间资产、候选包资产和正式商品资产。
- 销端读取资产是否需要签名 URL 或临时授权。

建议：

- 第一阶段文档层使用 `asset://` 表示逻辑引用。
- 实现前再确认实际对象存储和访问授权方案。

影响：

- 影响 `ProductPackage.assets`、`preview_assets`、下载记录和权益校验。

需要更新：

- [05-contracts.md](05-contracts.md)
- [08-contract-examples.md](08-contract-examples.md)
- [09-logical-data-model.md](09-logical-data-model.md)
- [10-interface-boundaries.md](10-interface-boundaries.md)

### 2.5 人工审核角色和权限

状态：open

需要确认：

- 创作者是否可以执行所有人工干预动作。
- 运营人员是否可以执行 `override`。
- 哪些动作需要记录风险说明。

建议：

- 创作者可以 `approve`、`edit`、`choose`、`rerun`。
- 运营人员可以 `reject`、`override`。
- `override` 必须记录操作者、原因和风险说明。

影响：

- 影响 `HumanReviewRecord`、任务恢复逻辑和审计要求。

需要更新：

- [06-pipeline-runtime.md](06-pipeline-runtime.md)
- [09-logical-data-model.md](09-logical-data-model.md)
- [10-interface-boundaries.md](10-interface-boundaries.md)

### 2.6 商品价格和支付验收范围

状态：proposed

建议：

- 第一阶段记录建议价格、模拟订单、基础销售记录和分成比例预留字段。
- 不接真实支付，不做完整结算。

影响：

- 影响销端验收范围。
- 影响 `OrderRecord` 和 `MarketFeedback.sales_summary` 的数据来源。

需要更新：

- [04-sales-platform.md](04-sales-platform.md)
- [09-logical-data-model.md](09-logical-data-model.md)
- [12-first-phase-acceptance-plan.md](12-first-phase-acceptance-plan.md)

### 2.7 反馈触发优化任务的门槛

状态：decided

已确认：

- 第一阶段不是所有 `MarketFeedback` 都自动触发优化任务。
- 反馈先进入运营确认。
- 低样本反馈只记录，不自动触发重做。

决策：

- 第一阶段由运营确认后创建优化任务。

影响：

- 影响反馈回流闭环的自动化程度。
- 影响产端是否需要反馈筛选规则。

已同步：

- [04-sales-platform.md](04-sales-platform.md)
- [10-interface-boundaries.md](10-interface-boundaries.md)
- [12-first-phase-acceptance-plan.md](12-first-phase-acceptance-plan.md)

### 2.8 契约版本策略落地方式

状态：decided

已确认：

- 契约版本独立于商品版本。
- 跨端对象使用可选 `schema_version` 字段。
- 第一阶段允许销端保留一个旧契约版本读取窗口。

决策：

- 第一阶段新增 `schema_version` 作为可选字段。
- 契约破坏性变更必须新增版本。

影响：

- 影响跨端兼容和后续演进。

已同步：

- [05-contracts.md](05-contracts.md)
- [08-contract-examples.md](08-contract-examples.md)

## 3. 已后置事项

以下事项不阻塞第一阶段：

- 完整支付网关接入。
- 完整分成结算和打款。
- 创作者店铺。
- 插件市场。
- 外部开放平台。
- 多团队复杂协作。
- 复杂推荐系统。
- 长视频批量生产。
- 多语言同步出版。

## 4. 主要风险

### 4.1 视频生成成本和稳定性

风险：

- 视频生成 provider 成本高、耗时长、失败率高。

缓解：

- 第一阶段先采用受控占位资产或短片段。
- 视频线作为第二验证路径。

### 4.2 契约过早复杂化

风险：

- 为未来市场、结算、插件做过多字段，拖慢第一阶段。

缓解：

- 跨端对象只保留第一阶段必需字段。
- 后续字段走可选扩展和版本兼容策略。

### 4.3 产端和销端边界被打穿

风险：

- 销端为了展示方便读取产端任务状态或中间资产。

缓解：

- 销端只能依赖 `ProductPackage`、`Listing` 和自有记录。
- 商品详情不读取 `ProductionTask`。

### 4.4 人工干预绕过质量门禁

风险：

- 为了快速上架，人工覆盖可能跳过最终质检。

缓解：

- 人工干预不能绕过最终商品质检。
- `override` 必须记录风险说明。

### 4.5 反馈无法归因

风险：

- 反馈未绑定具体商品版本，导致优化效果无法评估。

缓解：

- `MarketFeedback` 必须绑定 `product_id`、`package_id` 和 `version`。
- 购买和下载记录必须绑定购买时版本。

## 5. 最终评审清单

### 5.1 文档完整性

- [ ] 索引可以解释每份文档的阅读顺序。
- [ ] 领域词汇没有明显冲突。
- [ ] 跨端契约有必填字段、可选字段和状态枚举。
- [ ] 契约样例覆盖报告和视频商品包。
- [ ] 运行模型覆盖任务状态、节点状态、质量门禁、人工干预和版本规则。
- [ ] 切片拆解有依赖和验收标准。
- [ ] 验收计划覆盖主闭环和异常闭环。
- [ ] 待决事项已明确状态和影响范围。

### 5.2 边界一致性

- [ ] 供端不暴露 provider 凭据和 Adapter 内部细节。
- [ ] 产端不把内部任务状态暴露为销端展示依赖。
- [ ] 销端不修改 `WorkflowTemplate`。
- [ ] `ProductPackage.status` 和 `Listing.listing_status` 边界清楚。
- [ ] `MarketFeedback` 必须绑定商品版本。

### 5.3 第一阶段可落地性

- [ ] 报告线可以作为主验收路径。
- [ ] 视频线验收模式已确认。
- [ ] 供端最小 provider 范围已确认。
- [ ] 资产 URI 和访问授权方向已确认。
- [ ] 人工审核角色和权限已确认。
- [ ] 支付和结算简化范围已确认。
- [ ] 反馈触发优化任务的门槛已确认。

## 6. 决策记录模板

```text
决策编号：
决策标题：
状态：open / proposed / decided / deferred
日期：
参与人：

背景：

选项：

决定：

影响文档：

后续动作：
```
