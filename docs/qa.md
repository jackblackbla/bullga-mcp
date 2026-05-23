# Bullga MCP Agent QA

Last synced: **2026-05-23**.

Bullga MCP includes a fixed-case QA harness that checks whether agents select the right evidence tools for Korean finance questions. The purpose is not just connectivity; it catches reasoning regressions such as answering an ownership question without calling `get_ownership`.

## Fixed Case Files

The backend stores fixed regression cases under `backend-main/webapp/mcp_server/qa_cases/`:

- `agent_tool_calling_v1.yaml` — canonical Korean finance user-question cases.
- `company_pool_v1.yaml` — reusable company groups and codes.

Each case defines:

- a stable ID and category,
- a Korean question template,
- company groups to expand across,
- expected primary evidence tools,
- allowed helper tools,
- forbidden tools,
- final-answer requirements.

## Categories

| Category | Expected evidence behavior |
| --- | --- |
| `company_profile` | Use company detail/profile or stock wiki evidence for what the company does. Lookup tools are helpers only. |
| `ownership` | Use `get_ownership`; news or stock metadata alone is insufficient. |
| `product` | Use stock wiki/profile evidence to distinguish direct manufacturing, distribution, components, and indirect exposure. |
| `relationship` | Distinguish equity, supply chain, customer, affiliate, and thematic relationships with wiki/profile/ownership evidence. |
| `catalyst` | Verify price movement first with prices/chart, then compare news/disclosures. |
| `financial_risk` | Use derived indicators, statement values, or detailed statements for company-specific financial claims. |
| `gap_detection` | Demonstrate Bullga's structured synthesis against direct disclosure lookup, and state gaps honestly. |

## Runbook

Dry-run fixed cases inside the backend container:

```bash
docker compose exec -T webapp python manage.py mcp_qa   --case-file mcp_server/qa_cases/agent_tool_calling_v1.yaml   --company-pool-file mcp_server/qa_cases/company_pool_v1.yaml   --case-limit 10   --company-limit 3   --dry-run
```

Run against the hosted endpoint with a direct MCP API key:

```bash
MCP_QA_API_KEY=bg_mcp_xxx docker compose exec -T webapp python manage.py mcp_qa   --mcp-url https://mcp.bullga.ai/mcp   --case-file mcp_server/qa_cases/agent_tool_calling_v1.yaml   --company-pool-file mcp_server/qa_cases/company_pool_v1.yaml
```

The harness injects the key as `Authorization: Bearer bg_mcp_xxx` in the generated Claude MCP config.

## Known Failure Patterns

- Helper-only answer: the agent resolves a company with `get_stock_casual_info` but never calls the domain evidence tool.
- News-only catalyst: the agent cites news without verifying price movement.
- Theme overreach: the agent treats theme membership as a direct business relationship.
- Manufacturing overclaim: the agent calls a distributor or materials company a direct manufacturer.
- Runtime/tool-result failure: the right tool was selected but backend/DNS/API failure prevented answer completion.

## Current Guardrails

- Ownership cases fail without `get_ownership`.
- Catalyst cases require both price confirmation and candidate cause evidence.
- Financial-risk cases prefer `get_financial_statement_values` or `get_stock_derived_indicators` for narrow numeric evidence.
- Product/value-chain cases should separate direct manufacturing from indirect exposure.
- Bullga product-gap answers must cite actual Bullga tools instead of generic marketing claims.
