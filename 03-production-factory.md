# 产端框架讨论稿

> 本文档说明产端产品线和能力设计。流水线运行规则见 [06-pipeline-runtime.md](06-pipeline-runtime.md)，商品包契约见 [05-contracts.md](05-contracts.md)。

## 1. 产端定位

产端是灵枢平台的数字化商品工厂。

它负责把供端提供的 Token、模型和工具，组织成可执行、可干预、可质检、可复用的数字商品生产流水线。

第一阶段落地两条产品线：

- 深度调研报告
- 视频生成

两条产品线共用同一套流水线框架，但使用不同的智能体、工具、质检规则和产出格式。

## 2. 核心原则

产端的核心原则是：

> 流水线机制相同，智能体配置不同。

也就是说，平台不为每条产品线重做一套系统，而是提供统一的 Pipeline Harness。

不同产品线通过配置以下内容形成差异：

- Workflow Template
- Agent Set
- Tool Set
- Human Review Points
- Quality Gates
- Output Schema
- Product Package Rules

## 3. 统一流水线框架

统一流水线可以抽象为：

```text
需求输入
  ↓
任务规划
  ↓
生产执行
  ↓
过程质检
  ↓
人工干预
  ↓
成品生成
  ↓
成品质检
  ↓
商品打包
```

具体产品线可以对这些阶段进行扩展、跳过或循环。

任务状态、节点类型、重跑规则和版本生成规则见 [06-pipeline-runtime.md](06-pipeline-runtime.md)。

## 4. LangGraph 编排

产端建议采用 LangGraph 类图式编排模式。

每条流水线是一个图，而不是固定线性流程。

它需要支持：

- 节点执行
- 条件分支
- 循环重试
- 状态保存
- 人工中断与恢复
- 质检不通过后的回退
- 多智能体协作
- 节点级工具替换

典型执行逻辑：

```text
生成节点
  ↓
质检节点
  ↓
通过 → 下一节点
不通过 → 回退重做
不确定 → 人工审核
```

## 5. 智能体接口

智能体需要做成能力接口，而不是写死某个模型或 prompt。

建议抽象：

```text
Agent
- name
- capability
- input_schema
- output_schema
- run(context)
- evaluate(optional)
- fallback_agents(optional)
```

同时区分：

```text
AgentDefinition：智能体能力定义
AgentImplementation：智能体具体实现
```

例如：

```text
AgentDefinition = 脚本生成智能体
AgentImplementation A = GPT 实现
AgentImplementation B = Claude 实现
AgentImplementation C = 自研实现
```

工作流引用智能体能力，平台配置决定具体实现。

## 6. 人工干预

人工干预是一等流程节点。

建议支持以下动作：

```text
approve：通过
reject：驳回
edit：修改
choose：选择版本
rerun：要求重跑
override：人工覆盖
```

人工节点适合放在：

- 需求确认
- 方案确认
- 大纲确认
- 脚本确认
- 分镜确认
- 成品确认
- 上架前确认

## 7. 质量检查

产端需要内置质量检查，而不是在流程末尾补一个审核按钮。

质量检查分两类：

### 7.1 过程质检

检查单个节点产出是否合格。

例如：

- 检索资料是否可信
- 报告大纲是否完整
- 视频脚本是否符合目标
- 图片素材是否符合风格

### 7.2 成品质检

检查最终商品是否具备交付和销售条件。

例如：

- 内容完整性
- 版权风险
- 违规风险
- 格式正确性
- 预览资产完整性
- 商品信息完整性

质量门禁建议输出：

```text
pass：通过
manual_review：进入人工审核
rerun：退回指定节点
fail：终止生产
```

## 8. 深度调研报告生产线

深度调研报告生产线的目标是生成可信、结构化、可交付的深度报告。

参考流程：

```text
输入检索主题
  ↓
需求澄清
  ↓
检索计划生成
  ↓
多源资料检索
  ↓
资料筛选与可信度评估
  ↓
信息归纳
  ↓
报告结构生成
  ↓
报告写作
  ↓
事实校验
  ↓
人工审核
  ↓
报告成品打包
```

核心智能体：

- 需求澄清 Agent
- 检索规划 Agent
- 多源检索 Agent
- 资料评估 Agent
- 信息归纳 Agent
- 报告写作 Agent
- 事实校验 Agent
- 质量检查 Agent

## 9. 视频生成生产线

视频生成生产线的目标是生成可交付、可展示、可售卖的视频类数字商品。

参考流程：

```text
输入视频需求
  ↓
创意策划
  ↓
脚本生成
  ↓
分镜生成
  ↓
素材生成/选择
  ↓
视频片段生成
  ↓
配音/字幕/音乐
  ↓
剪辑合成
  ↓
视频质检
  ↓
人工审核
  ↓
视频成品打包
```

核心智能体：

- 创意策划 Agent
- 脚本 Agent
- 分镜 Agent
- 素材 Agent
- 视频生成 Agent
- 剪辑 Agent
- 字幕配音 Agent
- 视频质检 Agent

## 10. 产端输出

产端最终输出标准商品包：

```text
ProductPackage
- product_id
- product_type
- title
- description
- cover
- preview_assets
- final_assets
- author_id
- tags
- category
- version
- quality_score
- license_type
- suggested_price
- production_metadata
```

这个商品包可以进入销端，也可以导出给外部渠道。

本节只保留核心字段示意。正式字段、状态、资产引用和授权约束见 [05-contracts.md](05-contracts.md)。

## 11. 第一阶段建设重点

第一阶段建议优先做：

- Pipeline Harness
- LangGraph 工作流执行
- Agent 能力接口
- 工具调用接口
- 人工干预节点
- 质量门禁
- 深度调研报告生产线
- 视频生成生产线
- ProductPackage 输出

暂不优先做复杂协作、多产品线插件市场和高级自动路由。
