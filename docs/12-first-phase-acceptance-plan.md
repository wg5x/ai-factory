# 第一阶段验收计划

本文档定义灵枢平台第一阶段的验收口径。

它用于最终统一检查：哪些能力必须完成，哪些能力可以用受控样例或占位资产验证，哪些能力不进入第一阶段。

## 1. 验收目标

第一阶段验收目标是跑通数字商品生产闭环：

```text
供端提供工具能力和资源策略
  ↓
产端执行可恢复、可质检的生产流水线
  ↓
产端生成标准 ProductPackage
  ↓
销端展示、预览、下载和记录基础销售行为
  ↓
销端生成 MarketFeedback
  ↓
产端基于反馈经运营确认创建优化任务
```

验收重点不是交易规模、推荐效果或自动化程度，而是验证三端职责边界和核心契约是否成立。

## 2. 验收范围

### 2.1 Alpha 必须验收

Alpha 是第一阶段的最短可演示主链路，必须跑通报告生产到反馈回流。

- `ToolCapability`、`ResourceQuota`、`ToolInvocationPolicy` 样例可用。
- 生产任务状态流转可追踪。
- 节点执行记录可追踪。
- 工具调用成本和额度记录可追踪。
- 质量门禁可以输出 `pass`、`manual_review`、`rerun`、`fail`。
- 人工干预可以暂停并恢复流水线。
- 产端可以生成 `ready_for_listing` 的 `ProductPackage`。
- 销端可以接入 `ProductPackage` 并维护自己的上架状态。
- 销端可以生成绑定商品版本的 `MarketFeedback`。

### 2.2 Beta 必须验收

Beta 是第一阶段完整闭环，在 Alpha 基础上补齐优化任务和运营追踪。

- 产端可以基于 `MarketFeedback` 经运营确认后创建优化任务。
- 至少一个优化任务可以生成补丁版本或新主版本。
- 端到端追踪能回答 [09-logical-data-model.md](09-logical-data-model.md) 中列出的追踪问题。

### 2.3 可以简化验收

- 视频生成可以使用受控占位资产或短片段验证闭环，且不阻塞报告线主闭环验收。
- 销售支付可以使用模拟订单验证基础销售记录。
- 成本可以先使用估算值，不要求接入真实账单。
- 质量门禁可以先使用规则和人工审核混合方式。

### 2.4 不进入第一阶段

- 复杂实时计费。
- 完整支付和结算链路。
- 完整分成打款。
- 复杂推荐系统。
- 创作者店铺。
- 插件市场。
- 多团队复杂协作。
- 外部开放平台。

## 3. 验收环境假设

第一阶段验收可以使用以下受控环境：

```text
creator_001：内测创作者
operator_001：运营审核人员
user_001：消费者
research_report：报告产品线
video_generation：视频产品线
internal_market：内部销端渠道
```

验收数据可以来自：

- 人工准备的主题和需求。
- 受控工具 provider。
- 受控占位资产。
- 模拟销售和反馈数据。

## 4. 核心验收场景

### 4.1 报告生产闭环

输入：

```text
topic：AI Agent 在企业知识管理中的应用
target_audience：企业决策者
report_depth：standard
language：zh-CN
expected_format：pdf
```

期望过程：

1. 创建 `research_report` 生产任务。
2. 查询供端工具能力和额度。
3. 执行检索、资料评估、大纲生成、报告写作和事实校验。
4. 至少经过一次人工审核点。
5. 最终质量门禁通过。
6. 生成报告类 `ProductPackage`。
7. 销端接入并展示。
8. 消费者预览、购买或下载。
9. 销端生成 `MarketFeedback`。
10. 产端基于反馈经运营确认创建优化任务。

通过标准：

- 生产任务最终进入 `completed`。
- 商品包状态为 `ready_for_listing`。
- 销端上架状态为 `listed`。
- 购买或下载记录绑定 `product_id`、`package_id` 和 `version`。
- `MarketFeedback` 绑定同一个 `package_id` 和 `version`。

### 4.2 质量门禁回退

输入：

```text
触发条件：事实校验发现来源不足或引用缺失。
```

期望过程：

1. 质量门禁输出 `rerun`。
2. 指定 `target_node_for_rerun`。
3. 目标节点之后的自动节点重新执行。
4. 人工确认过的节点默认不自动废弃。

通过标准：

- 质量门禁结果记录 `reason` 和 `target_node_for_rerun`。
- 被重跑节点生成新的 `NodeExecution`。
- 重跑不覆盖上一次成功产物。

### 4.3 人工干预恢复

输入：

```text
触发条件：大纲审核节点进入 waiting_for_human。
```

期望过程：

1. 任务进入 `waiting_for_human`。
2. 人工执行 `edit` 或 `approve`。
3. 系统生成或保留新的 `output_ref`。
4. 流水线从恢复入口继续执行。

通过标准：

- `HumanReviewRecord` 记录操作者、动作、原因和恢复节点。
- `edit` 动作生成新的 `output_ref`。
- 人工干预不能绕过最终质量门禁。

### 4.4 工具 fallback

输入：

```text
触发条件：默认 search.web provider 超时。
```

期望过程：

1. 工具调用按策略重试。
2. 重试失败后调用 fallback provider。
3. 工具调用记录保留 provider、成本、错误和输出引用。

通过标准：

- `ToolInvocationRecord` 可追踪默认 provider 和 fallback provider。
- 成本记录绑定生产任务和节点执行。
- 产端只依赖最终 `output_ref`。

### 4.5 视频商品包闭环（可简化验证）

输入：

```text
topic：AI Agent 是什么
target_audience：非技术用户
duration_target：3 分钟
style：explainer
language：zh-CN
aspect_ratio：16:9
```

期望过程：

1. 创建 `video_generation` 生产任务。
2. 生成脚本、分镜和素材计划。
3. 使用真实短片段或受控占位资产完成视频商品包。
4. 生成视频类 `ProductPackage`。
5. 销端可以展示视频商品详情和预览片段。

通过标准：

- `type_metadata` 包含时长、分辨率、字幕语言和预览片段引用。
- 视频资产可通过 `asset_id` 和 `uri` 引用。
- 如果使用占位资产，必须在验收记录中标记。

## 5. 切片验收矩阵

| 切片 | 验收方式 | 阶段归属 |
| --- | --- | --- |
| 3.1 冻结 v0 契约样例 | 对照 `08-contract-examples.md` 检查对象样例 | Alpha |
| 3.2 定义第一阶段演示闭环 | 人工确认演示场景和非目标 | Alpha |
| 3.3 供端能力目录与默认调用策略 | 查询能力、额度和策略样例 | Alpha |
| 3.4 生产任务最小运行骨架 | 创建任务并查看状态流转 | Alpha |
| 3.5 工具节点调用闭环 | 触发一次工具调用和成本记录 | Alpha |
| 3.6 质量门禁闭环 | 覆盖 pass、manual_review、rerun、fail | Alpha |
| 3.7 人工干预暂停与恢复 | 触发人工审核并恢复 | Alpha |
| 3.8 ProductPackage 生成与版本规则 | 生成 v1 正式商品包 | Alpha；优化版本在 Beta |
| 3.9 深度调研报告生产线 MVP | 跑通报告生产和上架 | Alpha |
| 3.10 视频生成生产线 MVP | 跑通视频商品包闭环 | Beta 可简化 |
| 3.11 销端接入商品包并展示 | 展示列表和详情 | Alpha |
| 3.12 预览、下载与基础销售记录 | 记录购买或下载 | Alpha |
| 3.13 评分评论与 MarketFeedback 回流 | 生成并回传反馈 | Alpha |
| 3.14 反馈驱动优化任务 | 经运营确认从反馈创建优化任务 | Beta |
| 3.15 端到端追踪与运营查询 | 查询完整链路 | Beta，不要求完整仪表盘 |

## 6. 验收通过标准

Alpha 通过需要满足：

- 报告生产闭环完整跑通。
- 销端能展示和交付报告类商品。
- 市场反馈能绑定版本并回流产端。

Beta 通过需要满足：

- 至少一个反馈能经运营确认创建优化任务。
- 至少一个优化任务能生成补丁版本或新主版本。
- 视频生产线的验证范围已明确；如纳入当期演示，按确认范围跑通商品包闭环。
- 端到端追踪能回答 [09-logical-data-model.md](09-logical-data-model.md) 中列出的追踪问题。

## 7. 验收不通过标准

出现以下情况时，第一阶段不应视为通过：

- 商品包无法追踪到生产任务。
- 反馈无法绑定商品版本。
- 销端读取产端内部任务状态作为展示依赖。
- 人工干预可以绕过最终质检。
- 工具调用成本无法绑定节点执行。
- 已上架版本被原地覆盖，无法追踪历史购买版本。

## 8. 验收记录模板

```text
验收日期：
验收人员：
产品线：
生产任务 ID：
商品包 ID：
商品版本：
销端 listing ID：
MarketFeedback ID：

通过场景：
- 待填写

简化或占位能力：
- 待填写

发现问题：
- 待填写

待决事项：
- 待填写

结论：
通过 / 有条件通过 / 不通过
```
