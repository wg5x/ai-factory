# 产品线 MVP 规格

本文档定义第一阶段两条产品线的最小可验收规格。

它补充 [03-production-factory.md](03-production-factory.md)，重点说明每条产品线的输入、流程、工具能力、人工干预点、质量门禁和商品包要求。

## 1. 第一阶段产品线策略

第一阶段建议按以下优先级推进：

1. 深度调研报告生产线先行，作为主验证路径。
2. 视频生成生产线作为第二验证路径，先确认成本、资产策略和演示口径。

原因：

- 报告生产线更容易验证检索、写作、质检、人工干预、打包和反馈回流。
- 视频生产线成本更高，且依赖素材、配音、字幕、合成和版权检查，适合在主闭环跑通后推进。
- 第一阶段验收以报告线主闭环为阻塞项，视频线用于验证视频类商品包契约和资产策略。

## 2. 深度调研报告生产线 MVP

### 2.1 目标

创作者输入一个调研主题，系统生成可预览、可下载、可上架的报告类 `ProductPackage`。

### 2.2 输入

第一阶段最小输入：

```text
topic
target_audience
report_depth
language
expected_format
```

字段说明：

- `topic`：调研主题。
- `target_audience`：目标读者，例如企业决策者、产品经理、投资分析师。
- `report_depth`：报告深度，例如 brief、standard、deep。
- `language`：输出语言。
- `expected_format`：交付格式，第一阶段默认为 pdf。

### 2.3 Workflow Template

第一阶段建议模板：

```text
input_intake
  ↓
requirement_clarification
  ↓
search_plan
  ↓
source_search
  ↓
source_evaluation
  ↓
outline_generation
  ↓
outline_review
  ↓
report_writing
  ↓
fact_check
  ↓
final_quality_gate
  ↓
package_report
```

### 2.4 节点规格

| 节点 | 类型 | 说明 |
| --- | --- | --- |
| `input_intake` | `agent_node` | 解析主题、受众、深度和格式要求。 |
| `requirement_clarification` | `human_review` | 必要时确认范围、行业、地区和时间窗口。 |
| `search_plan` | `agent_node` | 生成检索关键词、来源类型和检索优先级。 |
| `source_search` | `tool_node` | 调用 `search.web` 或后续扩展的 `search.academic`。 |
| `source_evaluation` | `agent_node` | 筛选来源，标记可信度和引用价值。 |
| `outline_generation` | `agent_node` | 生成报告结构和章节目标。 |
| `outline_review` | `human_review` | 人工确认报告大纲。 |
| `report_writing` | `agent_node` | 生成报告正文。 |
| `fact_check` | `quality_gate` | 检查事实、引用、时间敏感信息和来源质量。 |
| `final_quality_gate` | `quality_gate` | 检查完整性、格式、版权风险和商品信息。 |
| `package_report` | `package_node` | 生成报告类 `ProductPackage`。 |

### 2.5 工具能力

第一阶段必需能力：

```text
search.web
text.generate
report.export
content.moderate
```

可后置能力：

```text
search.academic
chart.generate
data.extract
translation
```

### 2.6 人工干预点

第一阶段至少保留：

- 需求澄清：确认主题边界、受众和时间范围。
- 大纲审核：确认章节结构是否符合交付目标。
- 最终确认：在商品打包前确认是否允许进入最终质检。

人工动作：

```text
approve
edit
reject
rerun
override
```

### 2.7 质量门禁

过程质检：

- 来源是否足够。
- 来源是否可信。
- 大纲是否覆盖主题。
- 正文是否引用来源。

成品质检：

- 报告是否完整。
- 引用是否可追踪。
- 事实是否存在明显错误。
- 是否存在版权或合规风险。
- 预览资产和最终资产是否完整。

门禁结果：

```text
pass
manual_review
rerun
fail
```

### 2.8 ProductPackage 要求

报告类 `type_metadata` 第一阶段至少包含：

```text
page_count
format
outline_ref
citation_summary_ref
```

资产要求：

- 最终报告 PDF。
- 预览 PDF 或预览页图片。
- 封面图。
- 引用摘要或来源列表引用。

### 2.9 验收场景

最小验收场景：

```text
主题：AI Agent 在企业知识管理中的应用
受众：企业决策者
深度：standard
语言：中文
格式：pdf
```

验收结果：

- 生成一个 `ready_for_listing` 报告类商品包。
- 报告包含清晰目录和来源引用。
- 销端可以展示标题、简介、封面、预览页、页数、格式和下载文件。
- 市场反馈可以绑定该商品包版本回流产端。

## 3. 视频生成生产线 MVP

### 3.1 目标

创作者输入视频需求，系统生成可预览、可下载、可上架的视频类 `ProductPackage`。

第一阶段视频线需要先确认真实生成范围：

```text
mode_a：真实生成完整视频。
mode_b：真实生成脚本、分镜和部分片段，最终视频用占位资产合成。
mode_c：只跑通视频商品包闭环，不调用高成本视频生成。
```

第一阶段默认采用 `mode_b`。如果 provider、成本或素材授权暂不满足，可以降级到 `mode_c`，但必须在验收记录中标明。

### 3.2 输入

第一阶段最小输入：

```text
topic
target_audience
duration_target
style
language
aspect_ratio
```

字段说明：

- `topic`：视频主题。
- `target_audience`：目标观众。
- `duration_target`：目标时长。
- `style`：视频风格，例如 explainer、course、promo。
- `language`：旁白和字幕语言。
- `aspect_ratio`：画幅，例如 16:9 或 9:16。

### 3.3 Workflow Template

第一阶段建议模板：

```text
input_intake
  ↓
creative_brief
  ↓
script_generation
  ↓
script_review
  ↓
storyboard_generation
  ↓
asset_plan
  ↓
clip_generation_or_selection
  ↓
voice_and_subtitle
  ↓
edit_assembly
  ↓
video_quality_gate
  ↓
package_video
```

### 3.4 节点规格

| 节点 | 类型 | 说明 |
| --- | --- | --- |
| `input_intake` | `agent_node` | 解析视频主题、受众、时长和风格。 |
| `creative_brief` | `agent_node` | 生成创意简报和表达目标。 |
| `script_generation` | `agent_node` | 生成旁白脚本和画面说明。 |
| `script_review` | `human_review` | 人工确认脚本口径和风险。 |
| `storyboard_generation` | `agent_node` | 生成分镜和镜头列表。 |
| `asset_plan` | `agent_node` | 列出需要生成或选择的素材。 |
| `clip_generation_or_selection` | `tool_node` | 调用视频、图片或素材选择能力。 |
| `voice_and_subtitle` | `tool_node` | 生成配音和字幕。 |
| `edit_assembly` | `tool_node` | 合成预览视频或最终视频。 |
| `video_quality_gate` | `quality_gate` | 检查音画同步、字幕、时长、版权和风险。 |
| `package_video` | `package_node` | 生成视频类 `ProductPackage`。 |

### 3.5 工具能力

第一阶段目标能力：

```text
text.generate
image.generate
video.generate
voice.synthesize
subtitle.generate
content.moderate
```

可后置能力：

```text
video.edit
music.generate
image.upscale
translation
```

如果采用 `mode_b`，`video.generate` 可以先只生成短片段或使用受控占位资产。

报告线主闭环验收不要求以上视频能力全部真实接入。

### 3.6 人工干预点

第一阶段至少保留：

- 脚本审核：确认内容口径。
- 分镜审核：确认画面表达。
- 成品确认：确认是否进入最终质检和打包。

人工动作：

```text
approve
edit
choose
rerun
override
```

### 3.7 质量门禁

过程质检：

- 脚本是否符合主题和受众。
- 分镜是否覆盖脚本。
- 素材是否存在明显版权风险。
- 配音和字幕是否匹配。

成品质检：

- 视频是否可播放。
- 音画是否同步。
- 字幕是否完整。
- 时长和分辨率是否符合要求。
- 预览片段和最终资产是否完整。

### 3.8 ProductPackage 要求

视频类 `type_metadata` 第一阶段至少包含：

```text
duration_seconds
resolution
subtitle_languages
preview_clip_ref
```

资产要求：

- 最终视频文件。
- 预览片段。
- 封面图。
- 字幕文件或字幕嵌入信息。

### 3.9 验收场景

最小验收场景：

```text
主题：AI Agent 是什么
受众：非技术用户
时长：3 分钟
风格：explainer
语言：中文
画幅：16:9
```

验收结果：

- 生成一个 `ready_for_listing` 视频类商品包。
- 商品包包含最终视频或受控占位最终资产。
- 销端可以展示标题、简介、封面、预览片段、时长、分辨率和下载文件。
- 市场反馈可以绑定该商品包版本回流产端。

## 4. 两条产品线共用能力

### 4.1 共用 Pipeline Harness

两条产品线共用：

- 生产任务状态机。
- 节点执行记录。
- 工具调用机制。
- 质量门禁结果。
- 人工干预恢复机制。
- 商品包生成和版本规则。

### 4.2 差异配置

两条产品线差异来自：

- `WorkflowTemplate`。
- Agent Set。
- Tool Set。
- Quality Gates。
- Human Review Points。
- Output Schema。
- Product Package Rules。

## 5. 不进入第一阶段的能力

报告线暂不做：

- 自动生成复杂交互式图表。
- 自动采集付费数据库。
- 多语言同步出版。
- 深度专家协同审稿。

视频线暂不做：

- 长视频批量生产。
- 复杂多角色动画。
- 完整音乐版权市场。
- 多平台自动投放。

共用能力暂不做：

- 产品线插件市场。
- 智能自动路由优化。
- 复杂多团队协作。
- 完整创作者店铺体系。
