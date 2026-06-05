# 逻辑数据模型

本文档描述灵枢平台第一阶段的核心数据对象、归属边界和关联关系。

它不是数据库表结构设计，也不规定具体 ORM、索引或存储方案。

## 1. 模型原则

第一阶段数据模型遵循以下原则：

- 跨端对象以 [05-contracts.md](05-contracts.md) 为准。
- 端内对象可以独立演进，但不能泄露给其他端作为强依赖。
- 大文本、大文件和中间资产使用引用字段，不直接塞进任务状态或跨端对象。
- 所有可追踪对象都必须能关联到生产任务、商品版本或反馈窗口。
- 销端上架状态和产端商品包交付状态分开维护。

## 2. 供端对象

### 2.1 ToolCapability

归属：供端

用途：

- 描述产端可请求的抽象工具能力。
- 隔离具体 provider 差异。

关键关系：

- 一个 `ToolCapability` 可以被多个产品线使用。
- 一个 `ToolCapability` 可以匹配多个 `ToolInvocationPolicy`。

第一阶段状态：

```text
active
degraded
disabled
experimental
```

### 2.2 ToolProvider

归属：供端

用途：

- 描述具体工具供应商，例如搜索、文本生成、图片生成、视频生成 provider。

关键字段：

```text
provider_id
provider_name
provider_type
status
auth_ref
metadata
```

设计约束：

- `auth_ref` 指向安全凭据引用，不在业务对象中保存明文密钥。
- provider 状态异常时，供端通过 `ToolInvocationPolicy` fallback。

### 2.3 ToolAdapter

归属：供端

用途：

- 把具体 provider 的输入输出转换成统一能力接口。

关键关系：

- 一个 `ToolAdapter` 绑定一个 `ToolProvider`。
- 一个 `ToolAdapter` 可以服务一个或多个 `ToolCapability`。

第一阶段最小字段：

```text
adapter_id
provider_id
capability_id
input_schema
output_schema
status
fallback_adapter_id
```

### 2.4 ResourceQuota

归属：供端

用途：

- 描述某个主体在某类资源上的可用额度。

关键关系：

- `subject_type` 可以是创作者、产品线、工作流、工具能力或运营人员。
- 额度检查发生在节点执行前。

### 2.5 QuotaLedger

归属：供端

用途：

- 记录额度预留、消耗、退回和调整。

第一阶段最小字段：

```text
ledger_id
quota_id
production_task_id
node_execution_id
resource_type
change_type
amount
reason
created_at
```

`change_type` 可取值：

```text
reserve
consume
refund
adjust
```

### 2.6 ToolInvocationPolicy

归属：供端

用途：

- 描述 provider 选择、重试、额度、成本和限流策略。

关键关系：

- 一个策略绑定一个 `capability_id`。
- 一个策略通过 `scope.product_line_id` 约束适用产品线。

### 2.7 ToolInvocationRecord

归属：供端

用途：

- 记录一次工具调用的输入引用、输出引用、成本和错误。

第一阶段最小字段：

```text
tool_invocation_id
production_task_id
node_execution_id
capability_id
provider_id
adapter_id
input_ref
output_ref
status
cost_summary
error
started_at
finished_at
```

## 3. 产端对象

### 3.1 ProductLine

归属：产端

用途：

- 描述某类数字商品的生产配置集合。

第一阶段产品线：

```text
research_report
video_generation
```

关键关系：

- 一个 `ProductLine` 绑定一个或多个 `WorkflowTemplate`。
- 一个 `ProductLine` 拥有自己的 Agent Set、Tool Set、Quality Gates 和 Product Package Rules。

### 3.2 WorkflowTemplate

归属：产端

用途：

- 描述产品线的流程模板，包括节点、分支、回退关系、人工干预点和质量门禁位置。

第一阶段最小字段：

```text
workflow_template_id
product_line_id
version
nodes
edges
human_review_points
quality_gates
package_rules
status
```

设计约束：

- `WorkflowTemplate` 可以被市场反馈优化建议影响，但不能被销端直接修改。
- 破坏性修改应新增模板版本。

### 3.3 WorkflowNode

归属：产端

用途：

- 描述流水线中的节点定义。

节点类型：

```text
agent_node
tool_node
quality_gate
human_review
package_node
```

第一阶段最小字段：

```text
node_id
node_type
name
input_schema
output_schema
next_nodes
rerun_targets
config
```

### 3.4 ProductionTask

归属：产端

用途：

- 表示一次数字商品生产过程的运行实例。

第一阶段最小字段：

```text
production_task_id
product_line_id
workflow_template_id
creator_id
status
input_ref
task_kind
source_feedback_id
parent_production_task_id
current_node_id
created_at
updated_at
completed_at
failed_reason
```

`task_kind` 第一阶段可取值：

```text
new_product
optimization
```

设计约束：

- 基于市场反馈创建的优化任务必须记录 `source_feedback_id`。
- 优化任务应记录 `parent_production_task_id`，用于追踪它基于哪一次生产任务继续优化。
- 普通新商品生产任务的 `source_feedback_id` 和 `parent_production_task_id` 可以为空。

### 3.5 NodeExecution

归属：产端

用途：

- 记录一次节点执行。

字段以 [06-pipeline-runtime.md](06-pipeline-runtime.md) 的节点执行记录为准。

关键关系：

- 一个 `ProductionTask` 有多个 `NodeExecution`。
- 工具节点会关联一个或多个 `ToolInvocationRecord`。
- 人工节点会关联 `HumanReviewRecord`。
- 质量门禁节点会关联 `QualityGateResult`。

### 3.6 ArtifactRef

归属：产端

用途：

- 引用节点输入、节点输出、中间资产和最终资产。

第一阶段最小字段：

```text
artifact_ref
artifact_type
asset_scope
uri
checksum
created_by_node_execution_id
created_at
metadata
```

`asset_scope` 第一阶段可取值：

```text
intermediate
candidate_package
official_package
```

设计约束：

- `intermediate` 资产使用 `artifact://`，只供产端和运营审核视图使用。
- `candidate_package` 资产可以进入候选商品包，但不能交付销端。
- `official_package` 资产使用 `asset://`，可以进入正式 `ProductPackage`。
- 销端只能读取进入正式 `ProductPackage` 的 `asset://` 资产引用。

### 3.7 QualityGateResult

归属：产端

用途：

- 记录质量门禁结果。

第一阶段最小字段：

```text
quality_result_id
production_task_id
node_execution_id
decision
score
checked_items
reason
risk_flags
target_node_for_rerun
checked_at
```

### 3.8 HumanReviewRecord

归属：产端

用途：

- 记录人工干预动作和恢复入口。

第一阶段最小字段：

```text
review_id
production_task_id
node_execution_id
action
operator_id
operator_role
reason
risk_note
input_ref
output_ref
resume_node_id
created_at
```

设计约束：

- `creator` 角色可以执行 `approve`、`edit`、`choose`、`rerun`。
- `operator` 角色可以执行 `approve`、`edit`、`choose`、`rerun`、`reject`、`override`。
- `override` 必须记录 `risk_note`，并且不能绕过最终商品质检。

### 3.9 ProductPackageCandidate

归属：产端

用途：

- 表示生产任务完成前的候选商品包。

设计约束：

- 候选包不能交付销端上架。
- 候选包通过最终质检和授权检查后，才能生成正式 `ProductPackage`。

### 3.10 ProductPackage

归属：产端，跨端交付给销端

用途：

- 表示可交付给销端的标准商品包。

字段以 [05-contracts.md](05-contracts.md) 为准。

关键关系：

- 一个 `ProductPackage` 绑定一个 `ProductionTask`。
- 一个数字商品可以有多个 `ProductPackage` 版本。

## 4. 销端对象

### 4.1 Listing

归属：销端

用途：

- 表示商品包在销端的上架记录。

第一阶段最小字段：

```text
listing_id
product_id
package_id
version
listing_status
default_version
listed_at
delisted_at
```

`listing_status` 可取值：

```text
listed
delisted
archived
```

设计约束：

- `listing_status` 不写回 `ProductPackage.status`。
- 同一商品同一时间只能有一个默认展示版本。

### 4.2 Entitlement

归属：销端

用途：

- 表示用户对某个商品版本的购买或下载权益。

第一阶段最小字段：

```text
entitlement_id
user_id
product_id
package_id
version
source_order_id
created_at
```

### 4.3 OrderRecord

归属：销端

用途：

- 记录基础销售行为和分成预留字段。

第一阶段最小字段：

```text
order_id
user_id
product_id
package_id
version
amount
currency
payment_status
author_id
platform_share_ratio
creator_share_ratio
created_at
```

`payment_status` 第一阶段可取值：

```text
simulated_paid
free_download
```

设计约束：

- 第一阶段不接真实支付网关，`simulated_paid` 只用于验收购买记录和权益生成。
- `free_download` 用于免费样品或运营放行下载。

### 4.4 DownloadRecord

归属：销端

用途：

- 记录下载行为。

第一阶段最小字段：

```text
download_id
user_id
product_id
package_id
version
asset_id
downloaded_at
```

### 4.5 RatingRecord

归属：销端

用途：

- 记录评分。

第一阶段最小字段：

```text
rating_id
user_id
product_id
package_id
version
rating
created_at
```

### 4.6 CommentRecord

归属：销端

用途：

- 记录评论和可回流的用户需求。

第一阶段最小字段：

```text
comment_id
user_id
product_id
package_id
version
content
risk_status
created_at
```

### 4.7 MarketFeedback

归属：销端生成，产端消费

用途：

- 聚合一个时间窗口内的市场反馈。

字段以 [05-contracts.md](05-contracts.md) 为准。

## 5. 核心关联关系

```text
ProductLine
  └─ WorkflowTemplate
       └─ ProductionTask
            ├─ NodeExecution
            │    ├─ ToolInvocationRecord
            │    ├─ QualityGateResult
            │    └─ HumanReviewRecord
            └─ ProductPackage
                 └─ Listing
                      ├─ OrderRecord
                      ├─ DownloadRecord
                      ├─ RatingRecord
                      ├─ CommentRecord
                      └─ MarketFeedback
                           └─ Optimization ProductionTask
```

## 6. 状态归属

### 6.1 产端拥有的状态

```text
ProductionTask.status
NodeExecution.status
QualityGateResult.decision
ProductPackage.status
```

### 6.2 供端拥有的状态

```text
ToolCapability.status
ResourceQuota.status
ToolProvider.status
ToolAdapter.status
ToolInvocationRecord.status
```

### 6.3 销端拥有的状态

```text
Listing.listing_status
OrderRecord.payment_status
CommentRecord.risk_status
```

## 7. 第一阶段追踪问题

第一阶段至少要能回答以下问题：

- 某个商品包由哪个生产任务生成。
- 某个生产任务执行了哪些节点。
- 某个节点调用了哪些工具能力和 provider。
- 某个商品版本的质检结果是什么。
- 某个商品版本是否已上架。
- 某个用户购买或下载的是哪个商品版本。
- 某个商品版本收到哪些市场反馈。
- 某次优化任务来源于哪个 `MarketFeedback`。

## 8. 暂不建模

第一阶段暂不重点建模：

- 完整财务结算流水。
- 多店铺、多团队、多角色复杂协作。
- 复杂推荐系统特征。
- 插件市场安装关系。
- 面向外部开放平台的应用模型。
