# Bullga MCP

<p align="center">
  <img src="https://docs.bullga.ai/img/bullga-favicon-128.png" alt="Bullga logo" width="96" height="96">
</p>

<p align="center">
  AI-native access to structured and unstructured Korean financial market data.
</p>

**Endpoint:** `https://mcp.bullga.ai/mcp`

**Docs:** [Tool Surface](docs/tools.md) · [Access Control](docs/access-control.md) · [Agent QA](docs/qa.md) · [Hosted Overview](https://docs.bullga.ai/docs/intro)

**Project:** [Bullga.ai](https://bullga.ai) · [MCP Docs Repo](https://github.com/jackblackbla/bullga-mcp) · [Backend Source](https://github.com/jackblackbla/joah)

## What It Is

Bullga MCP is a hosted Model Context Protocol server for AI-native Korean finance workflows. It exposes Bullga's company, stock, disclosure, financial-statement, ownership, trading, theme, news, watchlist, and wiki data through Streamable HTTP at `/mcp`.

The MCP layer maps tools to Bullga backend REST/API views and returns structured JSON payloads that are suitable for agent reasoning, spreadsheet filling, report generation, and code workflows.

## Current Surface

Last synced: **2026-05-23** from the Bullga backend MCP registration list.

- **44 registered tools across 15 domains**.
- Direct access supports anonymous sessions, JWT session auth, and personal MCP API keys.
- Personal keys use the `bg_mcp_...` format and can be sent as `Authorization: Bearer <key>` or `X-API-Key: <key>`.
- Read tools are available under hourly and monthly credit limits. Higher-cost reads consume weighted credits.
- Watchlist tools require an authenticated Bullga user session or API key.
- Wiki write/admin tools (`wiki_ingest`, `wiki_file_answer`) require a Pro-or-higher authenticated user.
- The agent QA harness now uses fixed Korean finance cases so tool selection regressions are caught before release.

Recent quality updates reflected in this documentation:

- `get_disclosure` can hydrate missing disclosure bodies on demand from DART when server credentials are configured, and records `bodyFetchedAt` when body data is fetched.
- `list_news_articles` applies company filters only when the article explicitly mentions the target company name or alias, reducing stale association matches.
- Archived themes are kept out of live theme surfaces.
- Catalyst-style answers should verify price movement first, then compare news/disclosures; news alone is not sufficient evidence.

## Data Freshness

Bullga MCP is not a real-time trading feed. Market data is generally delayed unless a specific tool or upstream source states otherwise. Treat outputs as research and workflow context, not execution-grade market data.

## Connect

No local server install is required. Add the hosted endpoint to any MCP client that supports Streamable HTTP.

### Codex

```bash
codex mcp add bullga --url https://mcp.bullga.ai/mcp
codex mcp list
```

### Claude Code

```bash
claude mcp add --transport http bullga https://mcp.bullga.ai/mcp
```

Run `/mcp` inside Claude Code to confirm the server is connected and tools are available.

### Claude Desktop

Add Bullga under `mcpServers` in `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "bullga": {
      "url": "https://mcp.bullga.ai/mcp"
    }
  }
}
```

Restart Claude Desktop after editing the config.

### Optional API Key Header

If your client supports custom headers, attach a Bullga MCP API key for authenticated access and user-scoped quotas:

```http
Authorization: Bearer bg_mcp_xxx
```

or:

```http
X-API-Key: bg_mcp_xxx
```

## Authentication and Access

Bullga MCP supports two user-authenticated paths:

1. **Personal MCP API key** — preferred for remote clients and automation. Send the key as `Authorization: Bearer bg_mcp_xxx` or `X-API-Key: bg_mcp_xxx`.
2. **JWT session tools** — call `authenticate` once with Bullga access/refresh tokens; then use authenticated tools in the same MCP session. Call `refresh_access_token` when needed, or `logout` to clear the session.

Anonymous callers can use read tools within the Free policy limits. By default the Free policy allows 100 MCP calls per hour and 500 monthly MCP credits; Pro allows 1,000 calls per hour and 20,000 monthly credits; Enterprise limits are configurable and may be unlimited.

See [Access Control](docs/access-control.md) for plan defaults, weighted-credit behavior, and error responses.

## Company Lookup

Many company-linked tools accept `company_query`. You can pass a Korean company name, alias, stock code, DART corp code, or internal company ID depending on the tool. If the query is ambiguous, Bullga returns candidate companies instead of guessing.

Examples:

```json
{
  "tool": "get_ownership",
  "arguments": {
    "company_query": "삼성전자"
  }
}
```

```json
{
  "tool": "list_disclosures",
  "arguments": {
    "company_query": "이노와이어리스",
    "page_size": 5
  }
}
```

```json
{
  "tool": "list_news_articles",
  "arguments": {
    "company_query": "부산산업",
    "page_size": 5
  }
}
```

## Tool Coverage

| Domain | Tools | Use It For |
| --- | --- | --- |
| Auth | `authenticate`, `refresh_access_token`, `logout` | JWT session setup and teardown for session-authenticated workflows. |
| Companies | `list_companies`, `get_company` | Company search and canonical company identity lookup. |
| Convertible Bonds | `list_convertible_bonds` | Convertible bond issuance, conversion terms, and dilution context. |
| Disclosures | `list_disclosures`, `get_disclosure` | DART disclosure search and full disclosure body retrieval. |
| Financial Statements | `list_financial_statements`, `get_financial_statement`, `get_financial_statement_values`, `get_account_taxonomy_tree` | Statement listings, detailed statements, account taxonomy, and spreadsheet-style value extraction. |
| Investor Trading | `list_investor_trading` | KRX investor-type trading activity by stock/date. |
| Market Indices | `list_market_indices` | Korean market index OHLCV, value, and daily change records. |
| News | `list_news_articles` | Company-linked news listings for catalyst research. |
| Ownership | `get_ownership` | Major shareholders, related-party stakes, treasury shares, and voting-power estimates. |
| Short Selling | `list_short_selling_trades`, `list_short_selling_balances` | KRX short-selling trades and balance records. |
| Stock Wiki | `list_stock_wiki_index`, `get_stock_wiki_detail` | Bullga stock wiki index/detail pages for business and value-chain evidence. |
| Stocks | `list_stocks`, `get_stock_casual_info`, `get_stock_prices`, `get_stock_price_chart`, `get_stock_sparkline`, `get_stock_dividends`, `get_stock_derived_indicators` | Stock screening, casual stock info, price history, chart/sparkline data, dividends, and derived indicators. |
| Themes | `list_themes`, `get_theme`, `get_theme_members`, `get_theme_state_history`, `get_theme_index_history` | Public investment themes, members, confidence, index history, and state history. |
| Watchlists | `list_watchlists`, `create_watchlist`, `get_watchlist`, `update_watchlist`, `delete_watchlist`, `list_watchlist_items`, `add_watchlist_item`, `remove_watchlist_item` | Authenticated user watchlist management. |
| Wiki V2 | `wiki_get_page`, `wiki_ingest`, `wiki_lint`, `wiki_file_answer` | File-backed wiki page reads, ingest/lint operations, and cited answer filing. |

See [Tool Surface](docs/tools.md) for access notes and credit weights.

## Example Prompts

```text
Use list_companies to find 삼성전자, then use get_stock_casual_info for the primary listed security.
```

```text
Use get_ownership for 부산산업 and summarize the largest shareholder and related-party ownership.
```

```text
Use get_stock_price_chart, list_news_articles, and list_disclosures to explain whether 삼성전자가 최근 일주일간 오른 이유를 근거 있게 말할 수 있는지 확인해줘.
```

```text
Use get_financial_statement_values for 005930 and 000660 with columns 매출액, 영업이익, 자산, 부채.
```

## Agent QA

The backend includes a fixed-case MCP QA harness for Korean finance tool selection. It checks that agents call the narrow evidence tool for the user's intent, for example `get_ownership` for shareholder questions or price plus news/disclosure tools for catalyst questions.

See [Agent QA](docs/qa.md) for the runbook, expected tool rationale, and known failure patterns.

## Architecture

```text
MCP client
  -> Streamable HTTP /mcp
  -> FastMCP server
  -> access/quota wrapper
  -> Bullga Django REST/API views
  -> Bullga financial data stores and upstream market sources
```

Each sync tool closes Django database connections after invocation, and tool payloads mirror the backend's camelCased API response shapes.

## Notes

Bullga MCP provides data access tools for research and workflow automation. It does not provide investment advice. Validate data and assumptions before making financial decisions.
