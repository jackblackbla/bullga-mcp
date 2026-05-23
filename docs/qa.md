# Bullga MCP Agent QA

Last synced: **2026-05-23**.

Bullga MCP uses fixed Korean finance QA cases to check whether agents select the right evidence tools for a user's intent. The goal is to catch reasoning regressions, not just connectivity failures.

## What Each Case Defines

Each QA case includes:

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

## Expected Agent Behavior

Agents should:

- use the narrow evidence tool for the user's question,
- treat lookup tools as helpers rather than final evidence,
- verify market-move claims with price data before citing possible catalysts,
- state uncertainty when Bullga data does not support a claim,
- avoid overclaiming direct exposure from theme membership or indirect value-chain links.

## Known Failure Patterns

- Helper-only answer: the agent resolves a company with `get_stock_casual_info` but never calls the domain evidence tool.
- News-only catalyst: the agent cites news without verifying price movement.
- Theme overreach: the agent treats theme membership as a direct business relationship.
- Manufacturing overclaim: the agent calls a distributor or materials company a direct manufacturer.
- Tool-result failure: the right tool was selected but the answer does not preserve the error or uncertainty.

## Current Guardrails

- Ownership cases fail without `get_ownership`.
- Catalyst cases require both price confirmation and candidate cause evidence.
- Financial-risk cases prefer `get_financial_statement_values` or `get_stock_derived_indicators` for narrow numeric evidence.
- Product/value-chain cases should separate direct manufacturing from indirect exposure.
- Bullga product-gap answers must cite actual Bullga tools instead of generic marketing claims.
