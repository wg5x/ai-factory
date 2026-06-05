# 跨端对象契约

本文档定义供端、产端、销端之间传递的标准对象。

这些对象是三端解耦的 Interface。各端可以独立演进内部实现，但跨端通信必须遵守这里的契约。

## 1. 契约原则

1. 对象必须有稳定 `id`、`version` 或等价追踪字段。
2. 跨端对象只传递必要信息，不暴露内部实现细节。
3. 资产字段必须使用可追踪的引用，不直接传递不可管理的本地路径。
4. 状态字段必须可枚举，避免自由文本驱动流程。
5. 第一阶段字段可以轻量，但必须区分必填和可选。
6. 跨端对象建议携带可选 `schema_version`，用于区分契约版本和商品版本。

## 2. 供端到产端

### 2.1 ToolCapability

`ToolCapability` 描述产端可请求的工具能力。

必填字段：

```text
capability_id
name
input_schema
output_schema
supported_product_lines
status
```

可选字段：

```text
description
default_policy_id
quality_level
latency_level
cost_level
content_safety_level
metadata
```

`status` 可取值：

```text
active
degraded
disabled
experimental
```

设计约束：

- 产端只能依赖 `capability_id` 和 schema，不能依赖具体供应商。
- 供应商切换必须由供端完成。
- 具体 provider、重试、额度和限流策略不属于 `ToolCapability`，由 `ToolInvocationPolicy` 按调用范围解析。
- 第一阶段如需默认策略，可以使用可选的 `default_policy_id`，但产品线正式调用时仍应解析到明确策略。
- `experimental` 能力默认不能进入正式商品生产线。

### 2.2 ResourceQuota

`ResourceQuota` 描述某个主体在某类资源上的可用额度。

必填字段：

```text
quota_id
subject_type
subject_id
resource_type
total_limit
used_amount
reserved_amount
period
status
```

`subject_type` 可取值：

```text
creator
product_line
workflow
tool_capability
operator
```

`resource_type` 示例：

```text
token
image_generation
video_generation
search_request
storage
transcode
third_party_api
```

`status` 可取值：

```text
active
exhausted
suspended
expired
```

设计约束：

- 额度检查发生在节点执行前。
- 额度预留和实际消耗必须分开记录。
- 节点失败时是否退回额度，由 `ToolInvocationPolicy` 决定。
- `exhausted` 和 `suspended` 状态不能继续预留新额度。

### 2.3 ToolInvocationPolicy

`ToolInvocationPolicy` 描述工具调用策略。

必填字段：

```text
policy_id
capability_id
scope
allowed_subjects
default_provider
fallback_providers
retry_policy
quota_policy
cost_recording_policy
rate_limit_policy
```

`retry_policy` 至少包含：

```text
max_attempts
backoff
retryable_errors
```

`scope` 至少包含：

```text
product_line_id
```

`scope` 可选包含：

```text
workflow_id
subject_type
subject_id
```

设计约束：

- 第一阶段默认策略是 `默认工具 -> 失败重试 -> 备用工具`。
- 不同产品线可以共享能力，但使用不同策略。
- 成本记录必须绑定到生产任务和节点执行。

## 3. 产端到销端

### 3.1 ProductPackage

`ProductPackage` 是产端输出的标准商品包。

必填字段：

```text
product_id
package_id
product_type
title
description
author_id
version
status
assets
preview_assets
type_metadata
quality_result
license
production_task_id
created_at
updated_at
```

可选字段：

```text
cover
tags
category
suggested_price
pricing_currency
source_references
production_metadata
publish_channels
```

`status` 可取值：

```text
draft
quality_passed
ready_for_listing
archived
```

`type_metadata` 按 `product_type` 承载展示和交付需要的类型扩展信息。

报告类商品第一阶段至少包含：

```text
page_count
format
outline_ref
citation_summary_ref
```

视频类商品第一阶段至少包含：

```text
duration_seconds
resolution
subtitle_languages
preview_clip_ref
```

`assets` 至少包含：

```text
asset_id
asset_type
uri
checksum
format
size
license_status
```

`quality_result` 至少包含：

```text
score
decision
checked_items
risk_flags
reviewer
checked_at
```

设计约束：

- `status` 表示商品包交付状态，不表示生产任务状态。
- `listed`、`delisted` 属于销端上架状态，不属于 `ProductPackage.status`。
- 销端只能上架 `ready_for_listing` 状态的商品包。
- 产端可以生成多个版本，但同一时间销端只能把一个版本作为默认展示版本。
- 资产必须通过 `asset_id` 和 `uri` 引用，避免销端依赖产端内部存储结构。
- 正式商品包资产使用 `asset://` 逻辑引用；产端中间产物使用 `artifact://`，不得交付销端作为展示或下载依赖。
- 销端预览或下载时由资产服务把 `asset://` 解析为临时访问 URL，临时 URL 不写入跨端契约。
- `license_status` 未通过时不能进入正式销售。
- 生产任务完成前的打包产物只能作为候选包，不能交付销端上架。

## 4. 销端到产端

### 4.1 MarketFeedback

`MarketFeedback` 是销端回传给产端的市场反馈对象。

必填字段：

```text
feedback_id
product_id
package_id
version
event_window
metrics
created_at
```

可选字段：

```text
comments
refund_reasons
user_requests
improvement_suggestions
risk_reports
sales_summary
```

`metrics` 至少包含：

```text
views
downloads
purchases
rating_count
rating_avg
refund_count
complaint_count
```

设计约束：

- 销端回传反馈，不直接修改产端 Workflow Template。
- 产端可以基于反馈创建优化任务。
- 反馈必须绑定商品版本，否则无法判断优化是否有效。

## 5. 版本兼容

第一阶段采用轻量版本策略：

- `schema_version` 表示契约版本，`ProductPackage.version` 表示商品版本，两者不能混用。
- 契约新增可选字段不破坏兼容。
- 删除字段、修改字段含义、修改状态枚举必须视为破坏性变更。
- 破坏性变更需要新增契约版本。
- 销端和产端之间至少保留一个旧版本读取窗口。
