# 供端框架讨论稿

> 本文档说明供端内部职责。领域词汇见 [CONTEXT.md](CONTEXT.md)，供端输出对象契约见 [05-contracts.md](05-contracts.md)。

## 1. 供端定位

供端是灵枢平台的生产资料管理平台。

它负责管理数字商品生产所需要的 Token、模型额度、基础工具和第三方技术能力。

供端不直接生产商品，也不直接面向消费者售卖商品。它为产端提供稳定、统一、可替换的工具和资源供给。

## 2. 核心职责

供端主要承担四类职责：

1. 资源管理
2. 工具管理
3. 适配管理
4. 可用性与成本治理

## 3. 资源管理

资源主要包括：

- Token
- 模型调用额度
- 图片生成额度
- 视频生成额度
- 检索服务额度
- 存储与转码资源
- 第三方 API 配额

第一阶段不建议把 Token 做成用户侧强计费产品。

Token 更适合作为平台内部资源单位，用于：

- 统计生产成本
- 控制滥用
- 设置创作者额度
- 评估商品投入产出
- 决定是否继续补贴某类生产任务

用户侧更适合看到“可用生产能力”，而不是精确 Token 账单。

## 4. 工具能力目录

供端需要维护统一的工具能力目录。

示例能力包括：

```text
text.generate
text.rewrite
search.web
search.academic
image.generate
image.edit
image.upscale
video.generate
video.edit
voice.synthesize
subtitle.generate
report.export
content.moderate
```

产端调用的是能力，而不是具体供应商。

例如产端只请求：

```text
image.generate
```

供端再根据策略选择具体工具：

```text
Flux / DALL-E / 即梦 / Stable Diffusion / 其他新工具
```

## 5. 工具适配器

每个工具供应商通过 Tool Adapter 接入供端。

建议统一接口：

```text
ToolAdapter
- provider
- capability
- input_schema
- output_schema
- invoke()
- estimate_cost()
- health_check()
- fallback_provider
```

这样新技术出现时，只需要新增或替换 Adapter，不需要改动产端流水线。

Adapter 是供端内部实现细节；产端只依赖 `ToolCapability` 和 `ToolInvocationPolicy`。

## 6. 调度策略

供端需要支持基础调度策略：

- 按成本选择工具
- 按质量选择工具
- 按速度选择工具
- 按稳定性选择工具
- 按创作者等级选择工具
- 按产品线选择工具
- 按失败情况自动降级

第一阶段可以先实现简单策略：

```text
默认工具 → 失败重试 → 备用工具
```

后续再扩展为智能路由。

## 7. 权限与额度

供端需要给不同主体分配不同资源权限。

主体包括：

- 创作者
- 产品线
- 工作流
- 工具能力
- 内部运营人员

额度策略可以轻量开始：

- 新创作者基础额度
- 高质量创作者激励额度
- 高消耗低产出任务限制额度
- 内测产品线专项额度

## 8. 供端输出

供端向产端输出三类标准对象：

```text
ToolCapability
ResourceQuota
ToolInvocationPolicy
```

这些对象让产端知道：

- 能调用什么
- 谁可以调用
- 怎么调用
- 有多少额度
- 失败时怎么降级
- 成本如何记录

对象字段、状态和调用策略见 [05-contracts.md](05-contracts.md)。

## 9. 第一阶段建设重点

第一阶段建议优先做：

- 工具能力目录
- Token / 资源轻量记录
- 核心工具 Adapter
- 工具可用性监控
- 简单额度策略
- 产端调用接口

暂不优先做复杂套餐、实时计费、复杂账单、对外开放平台和插件市场。
