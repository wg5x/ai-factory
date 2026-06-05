# 产端流水线运行模型

本文档定义产端 Pipeline Harness 的运行规则。

它补充 [03-production-factory.md](03-production-factory.md)，重点说明生产任务如何执行、暂停、恢复、回退和生成商品版本。

## 1. 运行目标

Pipeline Harness 需要提供一套统一机制，让不同产品线复用同一执行框架。

第一阶段必须支持：

- 图式工作流执行。
- 节点级状态保存。
- 工具调用和成本记录。
- 人工干预暂停与恢复。
- 质量门禁回退。
- 商品包生成。
- 生产任务版本追踪。

## 2. 生产任务状态

生产任务建议使用以下状态：

```text
created
planning
running
waiting_for_human
quality_checking
packaging
completed
failed
cancelled
```

状态含义：

- `created`：任务已创建，尚未进入规划。
- `planning`：正在生成或确认执行计划。
- `running`：节点正在自动执行。
- `waiting_for_human`：任务等待人工处理。
- `quality_checking`：正在执行质量门禁。
- `packaging`：正在生成 `ProductPackage`。
- `completed`：已生成可交付商品包。
- `failed`：任务无法继续执行。
- `cancelled`：任务被人工取消。

设计约束：

- 只有 `completed` 状态可以生成正式 `ProductPackage`。
- `waiting_for_human` 必须记录等待原因和恢复入口。
- `failed` 必须记录失败节点、失败原因和可否重跑。

## 3. 节点类型

第一阶段节点类型建议控制在五类：

```text
agent_node
tool_node
quality_gate
human_review
package_node
```

### 3.1 agent_node

智能体节点。

用于需求澄清、检索规划、脚本生成、事实校验等智能处理。

### 3.2 tool_node

工具节点。

用于调用供端提供的工具能力，例如检索、图片生成、视频生成、导出等。

### 3.3 quality_gate

质量门禁节点。

用于检查节点产出或最终商品是否满足继续流转条件。

### 3.4 human_review

人工干预节点。

用于审核、编辑、选择版本、驳回、重跑或覆盖。

### 3.5 package_node

商品打包节点。

用于把最终资产、元信息、质检结果和授权信息组装成 `ProductPackage`。

## 4. 节点执行记录

每次节点执行都应生成执行记录。

必填字段：

```text
execution_id
production_task_id
node_id
node_type
status
input_ref
output_ref
started_at
finished_at
cost_summary
error
```

`status` 可取值：

```text
pending
running
succeeded
waiting_for_human
rerun_required
failed
skipped
```

设计约束：

- 节点输入输出应使用引用，而不是把大文本或大资产直接塞进任务状态。
- 成本记录必须能追踪到节点。
- 节点失败不能覆盖上一次成功产物。

## 5. 质量门禁结果

质量门禁统一输出：

```text
pass
manual_review
rerun
fail
```

处理规则：

- `pass`：进入下一节点。
- `manual_review`：任务进入 `waiting_for_human`。
- `rerun`：退回指定节点重新执行。
- `fail`：任务进入 `failed`。

质量门禁必须记录：

```text
checked_items
decision
reason
risk_flags
target_node_for_rerun
```

## 6. 人工干预动作

人工干预动作沿用产端框架中的定义：

```text
approve
reject
edit
choose
rerun
override
```

处理规则：

- `approve`：继续执行后续节点。
- `reject`：进入失败或回退指定节点。
- `edit`：保存人工修改后的产物，再继续执行。
- `choose`：在多个候选产物中选择一个作为后续输入。
- `rerun`：退回指定节点。
- `override`：记录人工覆盖原因后继续。

设计约束：

- `override` 必须记录操作者、原因和风险说明。
- 人工修改后的产物必须产生新的 `output_ref`。
- 人工节点不能直接绕过最终商品质检。

## 7. 回退与重跑

回退重跑需要满足三个条件：

1. 可以定位目标节点。
2. 可以找到目标节点需要的输入。
3. 可以判断哪些后续产物需要废弃或重新生成。

第一阶段建议采用保守策略：

- 重跑目标节点后，目标节点之后的自动节点全部重新执行。
- 人工确认过的节点默认不自动废弃，除非回退规则明确要求。
- 已上架商品不直接覆盖，必须生成新版本。

## 8. 商品版本生成

当生产任务完成并通过最终质量门禁后，产端生成 `ProductPackage`。

版本规则：

- 第一次完成生成 `v1`。
- 基于市场反馈优化生成 `v2`、`v3`。
- 同一版本的元信息修正可以生成补丁版本，例如 `v1.1`。
- 资产或内容发生实质变化时必须生成新主版本。

销端展示规则：

- 默认展示最新 `listed` 版本。
- 老版本可以保留下载或归档。
- 用户购买记录必须绑定购买时的商品版本。

## 9. 第一阶段最小闭环

第一阶段只需要跑通以下闭环：

```text
创建生产任务
  ↓
执行 Workflow Template
  ↓
调用供端工具能力
  ↓
过程质检
  ↓
必要时人工干预
  ↓
最终质检
  ↓
生成 ProductPackage
  ↓
销端接入展示
  ↓
MarketFeedback 回流
  ↓
创建优化任务
```

这个闭环跑通后，再扩展复杂协作、插件市场、智能路由和高级计费。

