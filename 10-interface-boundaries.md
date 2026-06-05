# 接口边界

本文档定义第一阶段供端、产端、销端之间的接口边界。

它只描述交互意图、请求响应和职责归属，不规定具体 REST、RPC、消息队列或 SDK 实现。

## 1. 边界原则

- 供端只暴露能力、额度、策略和工具调用结果，不暴露 provider 凭据和 Adapter 内部细节。
- 产端只暴露生产任务、商品包和优化任务入口，不把内部节点状态作为销端强依赖。
- 销端只消费 `ProductPackage`，只回传 `MarketFeedback`，不直接修改 `WorkflowTemplate`。
- 所有跨端接口必须使用 [05-contracts.md](05-contracts.md) 中定义的对象或对象引用。
- 大文本、大文件和中间资产通过引用传递。

## 2. 供端接口

### 2.1 查询工具能力

调用方：产端

目的：

- 获取某产品线可用的工具能力。

输入：

```text
product_line_id
creator_id
capability_names(optional)
```

输出：

```text
ToolCapability[]
```

约束：

- `experimental` 能力默认不返回给正式生产线。
- `disabled` 能力可以返回给运营视图，但不能被流水线调用。

### 2.2 查询额度

调用方：产端

目的：

- 在节点执行前检查主体是否还有可用额度。

输入：

```text
subject_type
subject_id
resource_type
product_line_id
workflow_id(optional)
```

输出：

```text
ResourceQuota[]
```

约束：

- `exhausted` 和 `suspended` 不能继续预留新额度。
- 额度展示给创作者时可以转译成“可用生产能力”，不直接展示精确 Token 账单。

### 2.3 解析工具调用策略

调用方：产端

目的：

- 根据能力、产品线和调用主体解析实际调用策略。

输入：

```text
capability_id
product_line_id
workflow_id(optional)
subject_type
subject_id
```

输出：

```text
ToolInvocationPolicy
```

约束：

- 同一个 `ToolCapability` 在不同产品线中可以解析到不同策略。
- 策略解析失败时，产端节点不能执行。

### 2.4 调用工具能力

调用方：产端

目的：

- 执行工具节点所需能力。

输入：

```text
production_task_id
node_execution_id
capability_id
policy_id
input_ref
```

输出：

```text
tool_invocation_id
status
output_ref
cost_summary
error
```

约束：

- 供端负责 provider 选择、重试和 fallback。
- 产端只依赖 `output_ref`，不依赖具体 provider 输出路径。
- 成本必须能追踪到 `production_task_id` 和 `node_execution_id`。

## 3. 产端接口

### 3.1 创建生产任务

调用方：创作者入口、销端优化入口或运营入口

目的：

- 启动一次数字商品生产。

输入：

```text
product_line_id
creator_id
input_ref
workflow_template_id(optional)
source_feedback_id(optional)
```

输出：

```text
production_task_id
status
current_node_id
```

约束：

- 基于市场反馈创建的优化任务必须记录 `source_feedback_id`。
- 如果没有指定 `workflow_template_id`，产端使用产品线默认模板。

### 3.2 查询生产任务

调用方：创作者入口、运营入口

目的：

- 查看生产任务状态和当前阻塞点。

输入：

```text
production_task_id
```

输出：

```text
production_task_id
status
current_node_id
waiting_reason
failed_reason
latest_package_id
updated_at
```

约束：

- 销端不应依赖该接口展示商品详情。
- 任务状态不等同于商品包状态。

### 3.3 提交人工干预

调用方：创作者入口、运营入口

目的：

- 对 `human_review` 节点进行审核、修改、选择版本或重跑。

输入：

```text
production_task_id
node_execution_id
action
operator_id
reason
output_ref(optional)
target_node_for_rerun(optional)
```

输出：

```text
review_id
production_task_id
status
resume_node_id
```

约束：

- `override` 必须记录原因和风险说明。
- `edit` 必须生成新的 `output_ref`。
- 人工干预不能绕过最终商品质检。

### 3.4 查询商品包

调用方：销端、运营入口

目的：

- 获取可交付给销端的 `ProductPackage`。

输入：

```text
package_id
```

输出：

```text
ProductPackage
```

约束：

- 只有正式 `ProductPackage` 可以返回给销端。
- 候选包只能在产端内部或运营审核视图中查看。

### 3.5 创建优化任务

调用方：销端、运营入口

目的：

- 基于 `MarketFeedback` 触发商品优化。
- 第一阶段建议由运营确认后调用，销端不自动直接触发重做。

输入：

```text
feedback_id
product_id
package_id
version
optimization_goal
```

输出：

```text
production_task_id
status
```

约束：

- 产端可以拒绝低置信度或数据不足的反馈。
- 优化任务必须生成新版本或明确记录“不生成新版本”的原因。

## 4. 销端接口

### 4.1 接入商品包

调用方：产端、运营入口

目的：

- 将 `ready_for_listing` 商品包导入销端商品目录。

输入：

```text
ProductPackage
```

输出：

```text
listing_id
listing_status
```

约束：

- 只接收 `ProductPackage.status = ready_for_listing` 的商品包。
- 销端上架状态不写回 `ProductPackage.status`。

### 4.2 查询商品列表

调用方：消费者入口

目的：

- 展示已上架商品。

输入：

```text
product_type(optional)
category(optional)
tag(optional)
```

输出：

```text
listing_id
product_id
package_id
version
title
description
cover
preview_summary
suggested_price
listing_status
```

约束：

- 第一阶段不做复杂推荐。
- 列表只展示 `listed` 状态。

### 4.3 查询商品详情

调用方：消费者入口

目的：

- 展示商品详情、预览资产和购买入口。

输入：

```text
listing_id
```

输出：

```text
ProductPackage fields for display
listing_status
preview_assets
type_metadata
```

约束：

- 展示字段来自 `ProductPackage` 和销端上架记录。
- 不读取产端内部任务状态或中间资产。

### 4.4 记录购买和下载

调用方：消费者入口

目的：

- 记录购买、下载和用户权益。

输入：

```text
user_id
product_id
package_id
version
asset_id
```

输出：

```text
order_id(optional)
download_id(optional)
entitlement_id
```

约束：

- 用户权益必须绑定购买时的 `package_id` 和 `version`。
- 第一阶段只记录基础销售数据，不做完整结算。

### 4.5 回传市场反馈

调用方：销端定时任务、运营入口

目的：

- 向产端回传一个时间窗口内的 `MarketFeedback`。

输入：

```text
MarketFeedback
```

输出：

```text
accepted
feedback_id
rejection_reason(optional)
```

约束：

- 反馈必须绑定 `product_id`、`package_id` 和 `version`。
- 销端不能通过反馈直接修改生产模板。

## 5. 端到端链路

### 5.1 报告生产到上架

```text
创作者提交调研主题
  ↓
产端创建 ProductionTask
  ↓
产端查询 ToolCapability / ResourceQuota / ToolInvocationPolicy
  ↓
产端执行 WorkflowTemplate
  ↓
工具节点通过供端调用 search.web / text.generate / report.export
  ↓
质量门禁通过
  ↓
产端生成 ProductPackage
  ↓
销端接入并创建 Listing
  ↓
消费者预览、购买、下载
```

### 5.2 反馈驱动优化

```text
消费者评分、评论、购买和下载
  ↓
销端聚合 MarketFeedback
  ↓
产端接收反馈
  ↓
产端创建优化 ProductionTask
  ↓
流水线重跑相关节点
  ↓
生成新版本 ProductPackage
  ↓
销端选择是否展示新版本
```

## 6. 错误边界

### 6.1 供端错误

示例：

```text
capability_disabled
quota_exhausted
policy_not_found
provider_unavailable
provider_timeout
```

处理原则：

- 可重试错误由供端按策略处理。
- 额度和权限错误直接返回产端，产端节点进入失败或等待人工处理。

### 6.2 产端错误

示例：

```text
workflow_template_not_found
node_execution_failed
quality_gate_failed
human_review_timeout
package_generation_failed
```

处理原则：

- 可恢复错误保留恢复入口。
- 不可恢复错误记录失败节点和原因。

### 6.3 销端错误

示例：

```text
package_not_ready_for_listing
asset_unavailable
listing_not_found
feedback_missing_version
```

处理原则：

- 销端不能修正产端商品包，只能拒绝接入或创建运营处理记录。
- 反馈缺少版本时不能回流产端。

## 7. 暂不定义

第一阶段暂不定义：

- 外部开放 API。
- 第三方插件接入接口。
- 完整支付网关接口。
- 完整财务结算接口。
- 复杂推荐和搜索排序接口。
