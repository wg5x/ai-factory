# 跨端契约样例

本文档为 [05-contracts.md](05-contracts.md) 中的跨端对象提供第一阶段样例。

样例用于统一供端、产端、销端的对接理解，不代表最终数据库结构。

## 1. 命名约定

第一阶段建议使用以下 ID 前缀：

```text
cap_：ToolCapability
quota_：ResourceQuota
policy_：ToolInvocationPolicy
task_：ProductionTask
node_exec_：NodeExecution
product_：数字商品
package_：ProductPackage
asset_：资产
feedback_：MarketFeedback
```

所有时间字段使用 ISO 8601 字符串。

## 2. ToolCapability 样例

### 2.1 search.web

```json
{
  "capability_id": "cap_search_web",
  "schema_version": "contract.v0",
  "name": "search.web",
  "input_schema": {
    "type": "object",
    "required": ["query"],
    "properties": {
      "query": { "type": "string" },
      "recency_days": { "type": "integer" },
      "domains": {
        "type": "array",
        "items": { "type": "string" }
      }
    }
  },
  "output_schema": {
    "type": "object",
    "required": ["results"],
    "properties": {
      "results": {
        "type": "array",
        "items": {
          "type": "object",
          "required": ["title", "url", "snippet"],
          "properties": {
            "title": { "type": "string" },
            "url": { "type": "string" },
            "snippet": { "type": "string" },
            "published_at": { "type": "string" }
          }
        }
      }
    }
  },
  "supported_product_lines": ["research_report"],
  "status": "active",
  "description": "Web search capability for research report production.",
  "default_policy_id": "policy_search_web_research_report_default",
  "quality_level": "standard",
  "latency_level": "medium",
  "cost_level": "low",
  "content_safety_level": "standard",
  "metadata": {
    "owner": "supply"
  }
}
```

### 2.2 text.generate

```json
{
  "capability_id": "cap_text_generate",
  "schema_version": "contract.v0",
  "name": "text.generate",
  "input_schema": {
    "type": "object",
    "required": ["prompt"],
    "properties": {
      "prompt": { "type": "string" },
      "style": { "type": "string" },
      "max_tokens": { "type": "integer" }
    }
  },
  "output_schema": {
    "type": "object",
    "required": ["text"],
    "properties": {
      "text": { "type": "string" },
      "usage": {
        "type": "object",
        "properties": {
          "input_tokens": { "type": "integer" },
          "output_tokens": { "type": "integer" }
        }
      }
    }
  },
  "supported_product_lines": ["research_report", "video_generation"],
  "status": "active",
  "description": "General text generation capability.",
  "default_policy_id": "policy_text_generate_default",
  "quality_level": "standard",
  "latency_level": "medium",
  "cost_level": "medium",
  "content_safety_level": "standard",
  "metadata": {
    "owner": "supply"
  }
}
```

## 3. ResourceQuota 样例

```json
{
  "quota_id": "quota_creator_001_token_monthly",
  "schema_version": "contract.v0",
  "subject_type": "creator",
  "subject_id": "creator_001",
  "resource_type": "token",
  "total_limit": 1000000,
  "used_amount": 120000,
  "reserved_amount": 30000,
  "period": "2026-06",
  "status": "active"
}
```

额度耗尽时：

```json
{
  "quota_id": "quota_creator_001_video_generation_monthly",
  "schema_version": "contract.v0",
  "subject_type": "creator",
  "subject_id": "creator_001",
  "resource_type": "video_generation",
  "total_limit": 60,
  "used_amount": 60,
  "reserved_amount": 0,
  "period": "2026-06",
  "status": "exhausted"
}
```

## 4. ToolInvocationPolicy 样例

```json
{
  "policy_id": "policy_search_web_research_report_default",
  "schema_version": "contract.v0",
  "capability_id": "cap_search_web",
  "scope": {
    "product_line_id": "research_report"
  },
  "allowed_subjects": [
    {
      "subject_type": "creator",
      "subject_tier": "internal_beta"
    }
  ],
  "default_provider": "provider_search_a",
  "fallback_providers": ["provider_search_b"],
  "retry_policy": {
    "max_attempts": 2,
    "backoff": "fixed_2s",
    "retryable_errors": ["timeout", "rate_limited", "provider_unavailable"]
  },
  "quota_policy": {
    "reserve_before_invoke": true,
    "refund_on_provider_failure": true,
    "refund_on_quality_failure": false
  },
  "cost_recording_policy": {
    "record_by": ["production_task_id", "node_execution_id", "capability_id"],
    "currency": "CNY"
  },
  "rate_limit_policy": {
    "per_subject_per_minute": 10,
    "per_product_line_per_minute": 100
  }
}
```

## 5. ProductPackage 样例

### 5.1 报告类商品包

```json
{
  "product_id": "product_research_ai_industry_001",
  "package_id": "package_research_ai_industry_001_v1",
  "schema_version": "contract.v0",
  "product_type": "research_report",
  "title": "AI Agent 行业深度调研报告",
  "description": "面向企业决策者的 AI Agent 行业趋势、应用场景和风险分析。",
  "author_id": "creator_001",
  "version": "v1",
  "status": "ready_for_listing",
  "assets": [
    {
      "asset_id": "asset_report_pdf_001",
      "asset_type": "final_report",
      "uri": "asset://reports/ai-agent-industry/v1/report.pdf",
      "checksum": "sha256:example",
      "format": "pdf",
      "size": 5242880,
      "license_status": "cleared"
    }
  ],
  "preview_assets": [
    {
      "asset_id": "asset_report_preview_001",
      "asset_type": "preview_pages",
      "uri": "asset://reports/ai-agent-industry/v1/preview.pdf",
      "checksum": "sha256:example-preview",
      "format": "pdf",
      "size": 1048576,
      "license_status": "cleared"
    }
  ],
  "type_metadata": {
    "page_count": 42,
    "format": "pdf",
    "outline_ref": "artifact://task_report_001/outline",
    "citation_summary_ref": "artifact://task_report_001/citations"
  },
  "quality_result": {
    "score": 88,
    "decision": "pass",
    "checked_items": ["completeness", "source_quality", "fact_check", "license"],
    "risk_flags": [],
    "reviewer": "quality_agent",
    "checked_at": "2026-06-05T08:00:00Z"
  },
  "license": {
    "license_type": "platform_standard",
    "commercial_use": true,
    "third_party_assets": []
  },
  "production_task_id": "task_research_ai_industry_001",
  "created_at": "2026-06-05T07:30:00Z",
  "updated_at": "2026-06-05T08:00:00Z",
  "cover": "asset://reports/ai-agent-industry/v1/cover.png",
  "tags": ["AI Agent", "行业研究", "企业应用"],
  "category": "research",
  "suggested_price": 199,
  "pricing_currency": "CNY",
  "source_references": ["artifact://task_report_001/source_list"],
  "production_metadata": {
    "product_line_id": "research_report",
    "workflow_template_id": "workflow_research_report_v0",
    "quality_gate_version": "qg_research_report_v0"
  },
  "publish_channels": ["internal_market"]
}
```

### 5.2 视频类商品包

```json
{
  "product_id": "product_video_ai_intro_001",
  "package_id": "package_video_ai_intro_001_v1",
  "schema_version": "contract.v0",
  "product_type": "video_generation",
  "title": "AI Agent 入门课程视频",
  "description": "面向非技术用户的 AI Agent 概念介绍和应用示例。",
  "author_id": "creator_001",
  "version": "v1",
  "status": "ready_for_listing",
  "assets": [
    {
      "asset_id": "asset_video_final_001",
      "asset_type": "final_video",
      "uri": "asset://videos/ai-agent-intro/v1/final.mp4",
      "checksum": "sha256:example-video",
      "format": "mp4",
      "size": 104857600,
      "license_status": "cleared"
    }
  ],
  "preview_assets": [
    {
      "asset_id": "asset_video_preview_001",
      "asset_type": "preview_clip",
      "uri": "asset://videos/ai-agent-intro/v1/preview.mp4",
      "checksum": "sha256:example-video-preview",
      "format": "mp4",
      "size": 10485760,
      "license_status": "cleared"
    }
  ],
  "type_metadata": {
    "duration_seconds": 180,
    "resolution": "1920x1080",
    "subtitle_languages": ["zh-CN"],
    "preview_clip_ref": "asset_video_preview_001"
  },
  "quality_result": {
    "score": 82,
    "decision": "pass",
    "checked_items": ["script_consistency", "audio_video_sync", "subtitle", "license"],
    "risk_flags": [],
    "reviewer": "quality_agent",
    "checked_at": "2026-06-05T09:00:00Z"
  },
  "license": {
    "license_type": "platform_standard",
    "commercial_use": true,
    "third_party_assets": []
  },
  "production_task_id": "task_video_ai_intro_001",
  "created_at": "2026-06-05T08:30:00Z",
  "updated_at": "2026-06-05T09:00:00Z"
}
```

## 6. MarketFeedback 样例

```json
{
  "feedback_id": "feedback_research_ai_industry_001_2026_06_05",
  "schema_version": "contract.v0",
  "product_id": "product_research_ai_industry_001",
  "package_id": "package_research_ai_industry_001_v1",
  "version": "v1",
  "event_window": {
    "from": "2026-06-05T00:00:00Z",
    "to": "2026-06-05T23:59:59Z"
  },
  "metrics": {
    "views": 1200,
    "downloads": 80,
    "purchases": 35,
    "rating_count": 12,
    "rating_avg": 4.3,
    "refund_count": 1,
    "complaint_count": 0
  },
  "created_at": "2026-06-06T00:10:00Z",
  "comments": [
    {
      "comment_id": "comment_001",
      "rating": 4,
      "content": "内容完整，但希望增加更多国内案例。",
      "created_at": "2026-06-05T12:00:00Z"
    }
  ],
  "refund_reasons": ["expected_more_case_studies"],
  "user_requests": ["增加国内企业案例", "补充 2026 年最新数据"],
  "improvement_suggestions": [
    {
      "source": "sales_summary_agent",
      "content": "建议在下一版本增加国内企业案例章节，并更新市场规模数据。"
    }
  ],
  "risk_reports": [],
  "sales_summary": {
    "gross_revenue": 6965,
    "currency": "CNY"
  }
}
```

## 7. 版本兼容样例

兼容新增字段：

```json
{
  "package_id": "package_research_ai_industry_001_v1",
  "schema_version": "contract.v0",
  "version": "v1",
  "status": "ready_for_listing",
  "new_optional_field": "allowed"
}
```

不兼容变更：

```text
删除 package_id。
把 status 的 ready_for_listing 改名为 ready。
把 version 从字符串改成不可兼容的数字结构。
```
