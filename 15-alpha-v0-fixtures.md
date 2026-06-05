# Alpha v0 冻结样例

本文档冻结 Alpha 开发所需的最小样例、报告线 `WorkflowTemplate` 和验收数据。

它用于研发排期、联调和验收，不代表最终数据库表结构。跨端对象字段仍以 [05-contracts.md](05-contracts.md) 为准，运行对象归属仍以 [09-logical-data-model.md](09-logical-data-model.md) 为准，接口交互仍以 [10-interface-boundaries.md](10-interface-boundaries.md) 为准。

## 1. Alpha 验收数据

```text
creator_id：creator_001
operator_id：operator_001
user_id：user_001
product_line_id：research_report
workflow_template_id：workflow_research_report_v0
channel_id：internal_market
topic：AI Agent 在企业知识管理中的应用
target_audience：企业决策者
report_depth：standard
language：zh-CN
expected_format：pdf
```

## 2. 已复用跨端样例

以下对象复用 [08-contract-examples.md](08-contract-examples.md)：

- `cap_search_web`
- `cap_text_generate`
- `quota_creator_001_token_monthly`
- `policy_search_web_research_report_default`

Alpha 还需要补充 `text.generate` 调用策略、`report.export`、`content.moderate`、报告线商品包、销端记录和市场反馈样例。

## 3. 供端补充样例

### 3.1 text.generate 调用策略

```json
{
  "policy_id": "policy_text_generate_default",
  "schema_version": "contract.v0",
  "capability_id": "cap_text_generate",
  "scope": {
    "product_line_id": "research_report"
  },
  "allowed_subjects": [
    {
      "subject_type": "creator",
      "subject_tier": "internal_beta"
    }
  ],
  "default_provider": "provider_text_generate_alpha",
  "fallback_providers": ["provider_text_generate_mock"],
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

### 3.2 report.export

```json
{
  "capability_id": "cap_report_export",
  "schema_version": "contract.v0",
  "name": "report.export",
  "input_schema": {
    "type": "object",
    "required": ["content_ref", "format"],
    "properties": {
      "content_ref": { "type": "string" },
      "format": { "type": "string", "enum": ["pdf"] },
      "cover_ref": { "type": "string" }
    }
  },
  "output_schema": {
    "type": "object",
    "required": ["asset_ref"],
    "properties": {
      "asset_ref": { "type": "string" },
      "page_count": { "type": "integer" },
      "checksum": { "type": "string" }
    }
  },
  "supported_product_lines": ["research_report"],
  "status": "active",
  "description": "Export structured report content to a PDF asset.",
  "default_policy_id": "policy_report_export_research_report_default",
  "quality_level": "standard",
  "latency_level": "medium",
  "cost_level": "low",
  "content_safety_level": "standard",
  "metadata": {
    "owner": "supply",
    "alpha_mode": "platform_adapter"
  }
}
```

```json
{
  "policy_id": "policy_report_export_research_report_default",
  "schema_version": "contract.v0",
  "capability_id": "cap_report_export",
  "scope": {
    "product_line_id": "research_report"
  },
  "allowed_subjects": [
    {
      "subject_type": "creator",
      "subject_tier": "internal_beta"
    }
  ],
  "default_provider": "provider_report_export_platform",
  "fallback_providers": ["provider_report_export_mock"],
  "retry_policy": {
    "max_attempts": 2,
    "backoff": "fixed_2s",
    "retryable_errors": ["timeout", "provider_unavailable"]
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
    "per_subject_per_minute": 5,
    "per_product_line_per_minute": 50
  }
}
```

### 3.3 content.moderate

```json
{
  "capability_id": "cap_content_moderate",
  "schema_version": "contract.v0",
  "name": "content.moderate",
  "input_schema": {
    "type": "object",
    "required": ["content_ref"],
    "properties": {
      "content_ref": { "type": "string" },
      "moderation_scope": {
        "type": "array",
        "items": { "type": "string" }
      }
    }
  },
  "output_schema": {
    "type": "object",
    "required": ["decision", "risk_flags"],
    "properties": {
      "decision": { "type": "string", "enum": ["pass", "manual_review", "fail"] },
      "risk_flags": {
        "type": "array",
        "items": { "type": "string" }
      },
      "reason": { "type": "string" }
    }
  },
  "supported_product_lines": ["research_report"],
  "status": "active",
  "description": "Moderate generated content with rule checks and manual-review fallback.",
  "default_policy_id": "policy_content_moderate_research_report_default",
  "quality_level": "standard",
  "latency_level": "medium",
  "cost_level": "low",
  "content_safety_level": "standard",
  "metadata": {
    "owner": "supply",
    "alpha_mode": "rule_and_human_mixed"
  }
}
```

```json
{
  "policy_id": "policy_content_moderate_research_report_default",
  "schema_version": "contract.v0",
  "capability_id": "cap_content_moderate",
  "scope": {
    "product_line_id": "research_report"
  },
  "allowed_subjects": [
    {
      "subject_type": "creator",
      "subject_tier": "internal_beta"
    }
  ],
  "default_provider": "provider_content_moderate_rules",
  "fallback_providers": ["provider_content_moderate_human_queue"],
  "retry_policy": {
    "max_attempts": 1,
    "backoff": "none",
    "retryable_errors": []
  },
  "quota_policy": {
    "reserve_before_invoke": false,
    "refund_on_provider_failure": false,
    "refund_on_quality_failure": false
  },
  "cost_recording_policy": {
    "record_by": ["production_task_id", "node_execution_id", "capability_id"],
    "currency": "CNY"
  },
  "rate_limit_policy": {
    "per_subject_per_minute": 20,
    "per_product_line_per_minute": 200
  }
}
```

## 4. 报告线 WorkflowTemplate v0

```json
{
  "workflow_template_id": "workflow_research_report_v0",
  "product_line_id": "research_report",
  "version": "v0",
  "status": "frozen",
  "nodes": [
    {
      "node_id": "input_intake",
      "node_type": "agent_node",
      "name": "输入解析",
      "next_nodes": ["requirement_clarification"],
      "config": {
        "agent_id": "agent_input_intake_v0"
      }
    },
    {
      "node_id": "requirement_clarification",
      "node_type": "human_review",
      "name": "需求澄清",
      "next_nodes": ["search_plan"],
      "config": {
        "allowed_actions": ["approve", "edit", "choose", "rerun"]
      }
    },
    {
      "node_id": "search_plan",
      "node_type": "agent_node",
      "name": "检索计划",
      "next_nodes": ["source_search"],
      "config": {
        "agent_id": "agent_search_plan_v0"
      }
    },
    {
      "node_id": "source_search",
      "node_type": "tool_node",
      "name": "资料检索",
      "next_nodes": ["source_evaluation"],
      "config": {
        "capability_id": "cap_search_web",
        "policy_id": "policy_search_web_research_report_default"
      }
    },
    {
      "node_id": "source_evaluation",
      "node_type": "agent_node",
      "name": "资料评估",
      "next_nodes": ["outline_generation"],
      "config": {
        "agent_id": "agent_source_evaluation_v0"
      }
    },
    {
      "node_id": "outline_generation",
      "node_type": "agent_node",
      "name": "大纲生成",
      "next_nodes": ["outline_review"],
      "config": {
        "agent_id": "agent_outline_generation_v0"
      }
    },
    {
      "node_id": "outline_review",
      "node_type": "human_review",
      "name": "大纲审核",
      "next_nodes": ["report_writing"],
      "config": {
        "allowed_actions": ["approve", "edit", "choose", "rerun", "reject"]
      }
    },
    {
      "node_id": "report_writing",
      "node_type": "agent_node",
      "name": "报告写作",
      "next_nodes": ["fact_check"],
      "config": {
        "agent_id": "agent_report_writing_v0",
        "capability_id": "cap_text_generate"
      }
    },
    {
      "node_id": "fact_check",
      "node_type": "quality_gate",
      "name": "事实校验",
      "next_nodes": ["final_review"],
      "rerun_targets": ["source_search", "source_evaluation", "report_writing"],
      "config": {
        "checked_items": ["source_quality", "citation_traceability", "time_sensitive_facts"]
      }
    },
    {
      "node_id": "final_review",
      "node_type": "human_review",
      "name": "最终人工确认",
      "next_nodes": ["package_report_candidate"],
      "config": {
        "allowed_actions_by_role": {
          "creator": ["approve", "edit", "choose", "rerun"],
          "operator": ["approve", "edit", "choose", "rerun", "reject", "override"]
        }
      }
    },
    {
      "node_id": "package_report_candidate",
      "node_type": "package_node",
      "name": "候选包打包",
      "next_nodes": ["final_quality_gate"],
      "config": {
        "asset_scope": "candidate_package",
        "capability_id": "cap_report_export",
        "policy_id": "policy_report_export_research_report_default"
      }
    },
    {
      "node_id": "final_quality_gate",
      "node_type": "quality_gate",
      "name": "最终质量门禁",
      "next_nodes": ["publish_report_package"],
      "rerun_targets": ["report_writing", "package_report_candidate"],
      "config": {
        "checked_items": ["completeness", "format", "license", "content_safety", "product_metadata"],
        "capability_id": "cap_content_moderate",
        "policy_id": "policy_content_moderate_research_report_default"
      }
    },
    {
      "node_id": "publish_report_package",
      "node_type": "package_node",
      "name": "正式商品包固化",
      "next_nodes": [],
      "config": {
        "asset_scope": "official_package",
        "product_package_status": "ready_for_listing"
      }
    }
  ],
  "edges": [
    ["input_intake", "requirement_clarification"],
    ["requirement_clarification", "search_plan"],
    ["search_plan", "source_search"],
    ["source_search", "source_evaluation"],
    ["source_evaluation", "outline_generation"],
    ["outline_generation", "outline_review"],
    ["outline_review", "report_writing"],
    ["report_writing", "fact_check"],
    ["fact_check", "final_review"],
    ["final_review", "package_report_candidate"],
    ["package_report_candidate", "final_quality_gate"],
    ["final_quality_gate", "publish_report_package"]
  ],
  "human_review_points": ["requirement_clarification", "outline_review", "final_review"],
  "quality_gates": ["fact_check", "final_quality_gate"],
  "package_rules": {
    "candidate_node": "package_report_candidate",
    "publish_node": "publish_report_package",
    "required_assets": ["final_report", "preview_pages", "cover"],
    "required_type_metadata": ["page_count", "format", "outline_ref", "citation_summary_ref"]
  }
}
```

## 5. 产端运行对象样例

### 5.1 ProductionTask

```json
{
  "production_task_id": "task_research_ai_km_001",
  "product_line_id": "research_report",
  "workflow_template_id": "workflow_research_report_v0",
  "creator_id": "creator_001",
  "status": "waiting_for_human",
  "input_ref": "artifact://task_research_ai_km_001/input",
  "task_kind": "new_product",
  "source_feedback_id": null,
  "parent_production_task_id": null,
  "current_node_id": "outline_review",
  "created_at": "2026-06-05T02:00:00Z",
  "updated_at": "2026-06-05T02:30:00Z",
  "completed_at": null,
  "failed_reason": null
}
```

### 5.2 NodeExecution

```json
{
  "execution_id": "node_exec_source_search_001",
  "production_task_id": "task_research_ai_km_001",
  "node_id": "source_search",
  "node_type": "tool_node",
  "status": "succeeded",
  "input_ref": "artifact://task_research_ai_km_001/search_plan",
  "output_ref": "artifact://task_research_ai_km_001/source_search/results",
  "started_at": "2026-06-05T02:12:00Z",
  "finished_at": "2026-06-05T02:13:10Z",
  "cost_summary": {
    "currency": "CNY",
    "estimated_amount": 0.2,
    "resource_usage": {
      "search_request": 3
    }
  },
  "error": null
}
```

```json
{
  "execution_id": "node_exec_outline_review_001",
  "production_task_id": "task_research_ai_km_001",
  "node_id": "outline_review",
  "node_type": "human_review",
  "status": "waiting_for_human",
  "input_ref": "artifact://task_research_ai_km_001/outline/draft",
  "output_ref": null,
  "started_at": "2026-06-05T02:28:00Z",
  "finished_at": null,
  "cost_summary": {
    "currency": "CNY",
    "estimated_amount": 0,
    "resource_usage": {}
  },
  "error": null
}
```

### 5.3 QualityGateResult

```json
{
  "quality_result_id": "qgr_fact_check_001",
  "production_task_id": "task_research_ai_km_001",
  "node_execution_id": "node_exec_fact_check_001",
  "decision": "rerun",
  "score": 62,
  "checked_items": ["source_quality", "citation_traceability", "time_sensitive_facts"],
  "reason": "引用来源数量不足，且缺少 2026 年最新案例。",
  "risk_flags": ["insufficient_sources"],
  "target_node_for_rerun": "source_search",
  "checked_at": "2026-06-05T03:05:00Z"
}
```

### 5.4 HumanReviewRecord

```json
{
  "review_id": "review_outline_001",
  "production_task_id": "task_research_ai_km_001",
  "node_execution_id": "node_exec_outline_review_001",
  "action": "edit",
  "operator_id": "creator_001",
  "operator_role": "creator",
  "reason": "补充国内企业知识库场景章节。",
  "risk_note": null,
  "input_ref": "artifact://task_research_ai_km_001/outline/draft",
  "output_ref": "artifact://task_research_ai_km_001/outline/creator_edit_001",
  "resume_node_id": "report_writing",
  "created_at": "2026-06-05T02:35:00Z"
}
```

```json
{
  "review_id": "review_final_override_001",
  "production_task_id": "task_research_ai_km_001",
  "node_execution_id": "node_exec_final_review_001",
  "action": "override",
  "operator_id": "operator_001",
  "operator_role": "operator",
  "reason": "允许进入候选包打包，但要求最终质量门禁重新检查授权和引用。",
  "risk_note": "部分来源仍需在 final_quality_gate 中重新校验。",
  "input_ref": "artifact://task_research_ai_km_001/report/final_draft",
  "output_ref": "artifact://task_research_ai_km_001/report/operator_override_001",
  "resume_node_id": "package_report_candidate",
  "created_at": "2026-06-05T03:30:00Z"
}
```

## 6. Alpha 商品包样例

```json
{
  "product_id": "product_research_ai_km_001",
  "package_id": "package_research_ai_km_001_v1",
  "schema_version": "contract.v0",
  "product_type": "research_report",
  "title": "AI Agent 在企业知识管理中的应用报告",
  "description": "面向企业决策者的 AI Agent 知识管理场景、落地路径和风险分析。",
  "author_id": "creator_001",
  "version": "v1",
  "status": "ready_for_listing",
  "assets": [
    {
      "asset_id": "asset_report_ai_km_pdf_001",
      "asset_type": "final_report",
      "uri": "asset://research_report/product_research_ai_km_001/v1/report.pdf",
      "checksum": "sha256:alpha-report-example",
      "format": "pdf",
      "size": 5242880,
      "license_status": "cleared"
    }
  ],
  "preview_assets": [
    {
      "asset_id": "asset_report_ai_km_preview_001",
      "asset_type": "preview_pages",
      "uri": "asset://research_report/product_research_ai_km_001/v1/preview.pdf",
      "checksum": "sha256:alpha-preview-example",
      "format": "pdf",
      "size": 1048576,
      "license_status": "cleared"
    }
  ],
  "type_metadata": {
    "page_count": 36,
    "format": "pdf",
    "outline_ref": "artifact://task_research_ai_km_001/outline/final",
    "citation_summary_ref": "artifact://task_research_ai_km_001/citations/summary"
  },
  "quality_result": {
    "score": 86,
    "decision": "pass",
    "checked_items": ["completeness", "format", "license", "content_safety", "product_metadata"],
    "risk_flags": [],
    "reviewer": "quality_agent",
    "checked_at": "2026-06-05T04:00:00Z"
  },
  "license": {
    "license_type": "platform_standard",
    "commercial_use": true,
    "third_party_assets": []
  },
  "production_task_id": "task_research_ai_km_001",
  "created_at": "2026-06-05T03:55:00Z",
  "updated_at": "2026-06-05T04:00:00Z",
  "cover": "asset://research_report/product_research_ai_km_001/v1/cover.png",
  "tags": ["AI Agent", "知识管理", "企业应用"],
  "category": "research",
  "suggested_price": 199,
  "pricing_currency": "CNY",
  "source_references": ["artifact://task_research_ai_km_001/source_list"],
  "production_metadata": {
    "product_line_id": "research_report",
    "workflow_template_id": "workflow_research_report_v0",
    "quality_gate_version": "qg_research_report_v0"
  },
  "publish_channels": ["internal_market"]
}
```

## 7. 销端对象样例

### 7.1 Listing

```json
{
  "listing_id": "listing_research_ai_km_001_v1",
  "product_id": "product_research_ai_km_001",
  "package_id": "package_research_ai_km_001_v1",
  "version": "v1",
  "listing_status": "listed",
  "default_version": true,
  "listed_at": "2026-06-05T04:10:00Z",
  "delisted_at": null
}
```

### 7.2 OrderRecord

```json
{
  "order_id": "order_research_ai_km_001_user_001",
  "user_id": "user_001",
  "product_id": "product_research_ai_km_001",
  "package_id": "package_research_ai_km_001_v1",
  "version": "v1",
  "amount": 199,
  "currency": "CNY",
  "payment_status": "simulated_paid",
  "author_id": "creator_001",
  "platform_share_ratio": 0.3,
  "creator_share_ratio": 0.7,
  "created_at": "2026-06-05T04:20:00Z"
}
```

### 7.3 Entitlement

```json
{
  "entitlement_id": "entitlement_research_ai_km_001_user_001",
  "user_id": "user_001",
  "product_id": "product_research_ai_km_001",
  "package_id": "package_research_ai_km_001_v1",
  "version": "v1",
  "source_order_id": "order_research_ai_km_001_user_001",
  "created_at": "2026-06-05T04:20:05Z"
}
```

### 7.4 DownloadRecord

```json
{
  "download_id": "download_research_ai_km_001_user_001",
  "user_id": "user_001",
  "product_id": "product_research_ai_km_001",
  "package_id": "package_research_ai_km_001_v1",
  "version": "v1",
  "asset_id": "asset_report_ai_km_pdf_001",
  "downloaded_at": "2026-06-05T04:21:00Z"
}
```

## 8. MarketFeedback 样例

```json
{
  "feedback_id": "feedback_research_ai_km_001_2026_06_05",
  "schema_version": "contract.v0",
  "product_id": "product_research_ai_km_001",
  "package_id": "package_research_ai_km_001_v1",
  "version": "v1",
  "event_window": {
    "from": "2026-06-05T00:00:00Z",
    "to": "2026-06-05T23:59:59Z"
  },
  "metrics": {
    "views": 120,
    "downloads": 8,
    "purchases": 3,
    "rating_count": 2,
    "rating_avg": 4.5,
    "refund_count": 0,
    "complaint_count": 0
  },
  "created_at": "2026-06-06T00:10:00Z",
  "comments": [
    {
      "comment_id": "comment_ai_km_001",
      "rating": 5,
      "content": "结构清楚，希望增加更多知识库迁移案例。",
      "created_at": "2026-06-05T12:00:00Z"
    }
  ],
  "refund_reasons": [],
  "user_requests": ["增加知识库迁移案例", "补充权限治理章节"],
  "improvement_suggestions": [
    {
      "source": "operator_001",
      "content": "下一版本补充知识库迁移案例和权限治理章节。"
    }
  ],
  "risk_reports": [],
  "sales_summary": {
    "gross_revenue": 597,
    "currency": "CNY"
  }
}
```

## 9. Alpha 冻结规则

- Alpha 开发只依赖本文档和 [08-contract-examples.md](08-contract-examples.md) 中的 v0 样例。
- 如果字段变化影响跨端对象，必须同步更新 [05-contracts.md](05-contracts.md) 和 [08-contract-examples.md](08-contract-examples.md)。
- 如果字段变化影响端内对象，必须同步更新 [09-logical-data-model.md](09-logical-data-model.md)。
- 如果节点顺序、人工点或质检点变化，必须同步更新 [11-product-line-mvp-specs.md](11-product-line-mvp-specs.md)。
