# 灵枢平台文档索引

本文档用于说明灵枢平台架构文档的阅读顺序、职责边界和第一阶段落地范围。

## 1. 阅读顺序

建议按以下顺序阅读：

1. [CONTEXT.md](CONTEXT.md)：统一领域词汇，避免同一概念在不同文档中被重复命名。
2. [01-overall-architecture.md](01-overall-architecture.md)：理解供端、产端、销端的整体分工。
3. [05-contracts.md](05-contracts.md)：理解三端之间通过哪些标准对象连接。
4. [08-contract-examples.md](08-contract-examples.md)：通过样例理解跨端对象如何落到第一阶段数据。
5. [09-logical-data-model.md](09-logical-data-model.md)：理解核心对象归属、关联关系和状态边界。
6. [10-interface-boundaries.md](10-interface-boundaries.md)：理解三端之间的接口边界和典型链路。
7. [06-pipeline-runtime.md](06-pipeline-runtime.md)：理解产端流水线如何执行、暂停、回退和生成版本。
8. [02-supply-platform.md](02-supply-platform.md)：理解供端资源、工具能力和 Adapter 设计。
9. [03-production-factory.md](03-production-factory.md)：理解产端产品线、智能体和质检设计。
10. [04-sales-platform.md](04-sales-platform.md)：理解销端展示、交易和反馈回流设计。
11. [11-product-line-mvp-specs.md](11-product-line-mvp-specs.md)：理解报告和视频产品线的 MVP 规格。
12. [07-first-phase-slices.md](07-first-phase-slices.md)：理解第一阶段如何拆成可排期、可验收的工作切片。
13. [12-first-phase-acceptance-plan.md](12-first-phase-acceptance-plan.md)：理解第一阶段如何最终验收。
14. [13-open-decisions.md](13-open-decisions.md)：查看进入开发前仍需确认的决策、风险和评审清单。
15. [14-development-readiness.md](14-development-readiness.md)：查看进入开发前的 Alpha / Beta 范围、mock 口径和开发就绪检查项。
16. [15-alpha-v0-fixtures.md](15-alpha-v0-fixtures.md)：查看 Alpha 开发冻结的 v0 样例、报告线 Workflow Template 和验收数据。
17. [16-alpha-implementation-plan.md](16-alpha-implementation-plan.md)：查看 Alpha 研发任务、依赖、并行建议和测试矩阵。
18. [17-alpha-issue-backlog.md](17-alpha-issue-backlog.md)：查看 Alpha 可发布为 issue 或 sprint item 的本地任务清单。
19. [18-alpha-acceptance-script-spec.md](18-alpha-acceptance-script-spec.md)：查看 Alpha 验收脚本入口、步骤、断言和证据输出。
20. [19-alpha-kickoff-checklist.md](19-alpha-kickoff-checklist.md)：查看 Alpha 开工前最后一次确认清单。
21. [PRD.md](PRD.md)：查看 Alpha 报告类数字商品主闭环的收束版 PRD。
22. [TODO.md](TODO.md)：查看 Alpha 开工、排期和验收用 TODO 清单。

## 2. 文档分工

### 总体架构文档

总体架构文档只回答三个问题：

- 平台为什么拆成供端、产端、销端。
- 三端各自承担什么职责。
- 第一阶段做什么、不做什么。

它不定义字段细节，也不定义运行时状态。

### 领域词汇文档

领域词汇文档定义平台内部必须稳定使用的业务概念。

当一个概念会被多个文档或多个模块共同使用时，优先放入 `CONTEXT.md`。

### 契约文档

契约文档定义三端之间传递的标准对象。

第一阶段最重要的对象是：

- `ToolCapability`
- `ResourceQuota`
- `ToolInvocationPolicy`
- `ProductPackage`
- `MarketFeedback`

这些对象是三端解耦的 Interface。

### 契约样例文档

契约样例文档为跨端对象提供第一阶段样例。

它重点回答：

- 每个跨端对象长什么样。
- 报告类商品包和视频类商品包如何表达。
- 市场反馈如何绑定商品版本。
- 哪些变更属于兼容或不兼容。

### 逻辑数据模型文档

逻辑数据模型文档描述核心对象的归属、关系和状态边界。

它不定义数据库表结构，也不规定具体存储方案。

### 接口边界文档

接口边界文档描述供端、产端、销端之间如何交互。

它不规定具体 REST、RPC、消息队列或 SDK 实现。

### 运行模型文档

运行模型文档定义产端 Pipeline Harness 的执行规则。

它重点回答：

- 一个生产任务有哪些状态。
- 节点如何输入、输出和失败。
- 人工干预如何暂停和恢复流水线。
- 质检不通过如何回退。
- 商品版本如何生成。

### 端内框架文档

端内框架文档分别说明供端、产端、销端内部职责。

它们不覆盖跨端契约，也不替代运行模型。

### 产品线 MVP 规格文档

产品线 MVP 规格文档定义报告和视频产品线第一阶段的最小输入、流程、工具、人工干预点、质量门禁和商品包要求。

### 切片拆解文档

切片拆解文档不重新定义架构、契约或运行规则。

它负责把第一阶段目标拆成可排期、可验收的工作切片，并说明切片之间的依赖关系。

### 验收计划文档

验收计划文档定义第一阶段的通过标准、核心验收场景和不通过标准。

它用于最终统一检查。

### 待决事项文档

待决事项文档集中记录进入开发前需要人工确认的问题、建议方案、风险和评审清单。

### 开发就绪文档

开发就绪文档汇总第一阶段进入开发前的最终口径。

它重点回答：

- Alpha 和 Beta 分别做什么。
- 哪些能力必须真实实现，哪些能力可以 mock 或简化。
- 进入开发前必须产出哪些样例和验收数据。
- 哪些检查项通过后可以开始开发。

### Alpha v0 冻结样例文档

Alpha v0 冻结样例文档提供 Alpha 开发和联调可直接使用的最小样例。

它重点回答：

- Alpha 使用哪组验收数据。
- 报告线 `WorkflowTemplate v0` 的节点、边、人工点和质量门禁是什么。
- Alpha 需要哪些产端运行对象和销端对象样例。
- Alpha 联调时一条商品链路的 ID 如何闭合。

### Alpha 实施计划文档

Alpha 实施计划文档把冻结样例转成可排期的研发任务。

它重点回答：

- Alpha 按什么顺序开发。
- 哪些任务可以并行，哪些任务必须串行。
- 每个任务的依赖和验收是什么。
- Alpha 需要覆盖哪些最小测试场景。

### Alpha 本地 Issue Backlog 文档

Alpha 本地 Issue Backlog 文档把实施计划转成可领取、可发布的任务项。

它重点回答：

- 哪些任务可以作为独立 issue 或 sprint item。
- 每个任务的依赖、建设内容和验收标准是什么。
- 发布正式 issue 时建议按什么顺序排。

### Alpha 验收脚本规格文档

Alpha 验收脚本规格文档定义进入开发后需要实现的验收入口。

它重点回答：

- 每个验收脚本覆盖哪些研发任务。
- 脚本应执行哪些步骤和断言。
- 验收失败时需要输出哪些定位证据。

### Alpha 开工清单文档

Alpha 开工清单文档用于开始写代码前的最终确认。

它重点回答：

- 开工前哪些 owner、样例、环境和任务必须就绪。
- 哪些口径已经冻结，不能在开发中反复漂移。
- 哪些内容明确不进入 Alpha 开发。

### Alpha PRD 文档

Alpha PRD 文档把已有架构、契约、验收和开工口径收束为产品需求。

它重点回答：

- Alpha 为什么做。
- Alpha 具体做什么、不做什么。
- 谁使用这条主闭环。
- 完成后如何判定成功。

### Alpha TODO List 文档

Alpha TODO List 文档把 PRD 和实施计划转成开工、排期和验收用待办清单。

它重点回答：

- 开工前有哪些 gate。
- A0-A12 每个任务要完成哪些事项。
- 每个任务对应哪些验收脚本。
- Alpha 最终 done definition 是什么。

## 3. 第一阶段原则

第一阶段的目标不是一次性做完整交易市场，而是先验证数字商品生产闭环。

优先级从高到低为：

1. 供端工具能力可被稳定调用。
2. 产端可以执行可恢复、可质检的生产流水线。
3. 产端输出标准商品包。
4. 销端可以展示、交付并回传反馈。

暂不优先做：

- 复杂实时计费。
- 完整分成结算。
- 创作者店铺。
- 复杂推荐系统。
- 插件市场。
- 多团队复杂协作。

## 4. 文档维护规则

新增或修改设计时，按以下顺序判断应该改哪里：

1. 是领域名词变化，改 `CONTEXT.md`。
2. 是跨端对象变化，改 `05-contracts.md`。
3. 是跨端对象样例变化，改 `08-contract-examples.md`。
4. 是对象归属、关系或状态边界变化，改 `09-logical-data-model.md`。
5. 是三端接口边界变化，改 `10-interface-boundaries.md`。
6. 是产端执行规则变化，改 `06-pipeline-runtime.md`。
7. 是某一端内部职责变化，改对应端的框架文档。
8. 是产品线 MVP 规格变化，改 `11-product-line-mvp-specs.md`。
9. 是第一阶段排期、依赖或验收边界变化，改 `07-first-phase-slices.md` 或 `12-first-phase-acceptance-plan.md`。
10. 是进入开发前的待决事项、风险或评审清单变化，改 `13-open-decisions.md`。
11. 是开发前范围、mock 口径或就绪检查变化，改 `14-development-readiness.md`。
12. 是 Alpha v0 样例、报告线模板或验收数据变化，改 `15-alpha-v0-fixtures.md`。
13. 是 Alpha 任务拆解、依赖、测试矩阵或实施顺序变化，改 `16-alpha-implementation-plan.md`。
14. 是 Alpha 本地 issue、任务发布顺序或任务验收口径变化，改 `17-alpha-issue-backlog.md`。
15. 是 Alpha 验收脚本入口、步骤、断言或证据输出变化，改 `18-alpha-acceptance-script-spec.md`。
16. 是 Alpha 开工 owner、环境、任务发布或 code-start gate 变化，改 `19-alpha-kickoff-checklist.md`。
17. 是 Alpha 产品目标、用户故事、范围或成功标准变化，改 `PRD.md`。
18. 是 Alpha 待办事项、执行顺序或完成定义变化，改 `TODO.md`。
19. 是重大架构取舍，后续应新增 ADR 记录。
