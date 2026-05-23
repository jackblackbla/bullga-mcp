# Bullga MCP

<p align="center">
  <img src="https://docs.bullga.ai/img/bullga-favicon-128.png" alt="Bullga logo" width="96" height="96">
</p>

<p align="center">
  AI-native access to structured and unstructured Korean financial market data.
</p>

<p align="center">
  <a href="https://docs.bullga.ai/docs/intro">English</a> ·
  <a href="https://docs.bullga.ai/ko/docs/intro">한국어</a>
</p>

**Endpoint:** `https://mcp.bullga.ai/mcp`

**Docs:** [Overview](https://docs.bullga.ai/docs/intro) · [Getting Started](https://docs.bullga.ai/docs/getting-started) · [Authentication](https://docs.bullga.ai/docs/authentication) · [Architecture](https://docs.bullga.ai/docs/architecture) · [Tool Reference](https://docs.bullga.ai/docs/tools/company-lookup)

**Project:** [Bullga.ai](https://bullga.ai) · [GitHub](https://github.com/geonwoo/joah)

## What It Is

Bullga MCP is a hosted Model Context Protocol server built for AI-native financial workflows. It exposes both structured datasets, such as companies, prices, disclosures, financial statements, ownership, and trading activity, and unstructured source material, such as disclosure bodies, news, stock wiki pages, and file-backed LLM wiki content.

It runs over Streamable HTTP at `/mcp`, returns structured JSON payloads, and maps MCP tools to Bullga backend REST API endpoints.

Use it when an AI agent needs to look up Korean listed companies, stocks, disclosures, financial statements, investor trading, short selling, themes, watchlists, or Bullga wiki content without building a custom API integration.

## Data Freshness

Bullga MCP is not a real-time trading feed. Market data is generally delayed by 30 minutes unless a tool or upstream source states otherwise. Treat outputs as research and workflow context, not execution-grade market data.

## Use With AI Agents

The main advantage of Bullga MCP is that the data layer is already agent-ready. MCP-compatible agents can call Bullga directly from the environment where analysis and code work happen, including Codex, Claude Code, Claude Desktop, and Cursor.

There is no custom SDK, scraper, or one-off API wrapper to write first. Paste the endpoint into your MCP client, then ask the agent to call tools such as `list_companies`, `get_stock_prices`, `list_disclosures`, or `wiki_query`.

## Connect

No local server install is required. Add the hosted endpoint to any MCP client that supports Streamable HTTP.

### Codex

```bash
codex mcp add bullga --url https://mcp.bullga.ai/mcp
```

Verify it was added:

```bash
codex mcp list
```

### Claude Code

```bash
claude mcp add --transport http bullga https://mcp.bullga.ai/mcp
```

Then run `/mcp` inside Claude Code to confirm the server is connected and tools are available.

### Claude Desktop

#### One-Prompt Setup

Copy the prompt below into Claude Desktop. It contains the exact config edit needed to add Bullga MCP.

```text
Add the Bullga MCP server to my Claude Desktop configuration.

Edit the file claude_desktop_config.json:
- macOS: ~/Library/Application Support/Claude/claude_desktop_config.json
- Windows: %APPDATA%\Claude\claude_desktop_config.json

Add this MCP server entry. If the file already exists with other mcpServers, merge this in without removing existing entries:

{
  "mcpServers": {
    "bullga": {
      "url": "https://mcp.bullga.ai/mcp"
    }
  }
}

After editing, tell me to restart Claude Desktop for the changes to take effect.
```

Restart Claude Desktop after the config is updated. The Bullga tools will appear automatically in your next conversation.

#### Manual Setup

Add Bullga under `mcpServers` in your Claude Desktop config:

```json
{
  "mcpServers": {
    "bullga": {
      "url": "https://mcp.bullga.ai/mcp"
    }
  }
}
```

Restart Claude Desktop, then ask the client to use a Bullga tool.

### Cursor

In Cursor, open **Settings -> MCP**, add a new server, and use:

| Field | Value |
| --- | --- |
| Name | `bullga` |
| Transport | `streamable-http` |
| URL | `https://mcp.bullga.ai/mcp` |

## Quick Test

Ask your MCP client:

```text
Use the list_companies tool with search term "삼성전자".
```

If the connection is working, the client should return matching companies with identifiers such as name, stock code, sector, and related company metadata.

## Authentication

Most Bullga MCP tools work without authentication. Authentication is required only for Watchlists tools.

The server uses session-based JWT authentication:

1. Get an access token and refresh token from Bullga.
2. Call `authenticate` once in the MCP session.
3. Use watchlist tools in the same session without passing tokens again.
4. Call `refresh_access_token` when needed, or `logout` to clear the session.

Tools outside Watchlists, including Companies, Stocks, Disclosures, Financial Statements, Themes, News, Market Indices, and Wiki V2, can be called without an authenticated Bullga session.

## Company Lookup

Many company-linked tools accept `company_query`. You can pass a company name, alias, DART corp code, or stock code, and Bullga resolves it before calling the underlying data endpoint.

Examples:

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
  "tool": "wiki_query",
  "arguments": {
    "company_query": "삼성전기",
    "question": "What are the main products and supply chain risks?"
  }
}
```

If the query is ambiguous, Bullga returns candidate companies instead of guessing. If no match exists, it returns a not-found response.

## Tool Coverage

Bullga MCP currently exposes 47 tools across 15 domains.

| Domain | Tools | Use It For |
| --- | --- | --- |
| Auth | `authenticate`, `refresh_access_token`, `logout` | Session JWT setup for authenticated watchlist workflows. |
| Companies | `list_companies`, `get_company`, `get_company_business_profile` | Company master data, business profile summaries, sector and industry filters. |
| Convertible Bonds | `list_convertible_bonds` | Korean listed-company CB issuance, conversion terms, dilution metrics, and source filings. |
| Disclosures | `list_disclosures`, `get_disclosure` | DART disclosure search, metadata, and full disclosure body retrieval. |
| Financial Statements | `list_financial_statements`, `get_financial_statement`, `get_financial_statement_values`, `get_account_taxonomy_tree` | Parsed statements, account taxonomy, and spreadsheet-ready financial values. |
| Investor Trading | `list_investor_trading` | KRX investor-type trading volume and value by stock and date. |
| Market Indices | `list_market_indices` | Korean market index OHLCV, value, and daily change records. |
| News | `list_news_articles` | Company-linked news article listings. |
| Ownership | `get_ownership` | Major shareholders, related-party stakes, treasury shares, and voting-power estimates. |
| Short Selling | `list_short_selling_trades`, `list_short_selling_balances` | KRX short-selling trades and balance records. |
| Stock Wiki | `list_stock_wiki_index`, `get_stock_wiki_detail` | Bullga stock wiki pages, completeness, sections, and version state. |
| Stocks | `list_stocks`, `get_stock`, `get_stock_prices`, `get_stock_price_chart`, `get_stock_sparkline`, `get_stock_dividends`, `get_stock_derived_indicators`, `get_stock_shareholders` | Stock search, details, price history, chart data, dividends, derived indicators, and shareholder stats. |
| Themes | `list_themes`, `get_theme`, `get_theme_members`, `get_theme_index_history`, `get_theme_state_history` | Public investment themes, members, confidence, index history, and state history. |
| Watchlists | `list_watchlists`, `get_watchlist`, `create_watchlist`, `update_watchlist`, `delete_watchlist`, `list_watchlist_items`, `add_watchlist_item`, `remove_watchlist_item` | Authenticated user watchlist management. |
| Wiki V2 | `wiki_get_page`, `wiki_query`, `wiki_ingest`, `wiki_lint`, `wiki_file_answer` | File-backed LLM wiki pages, grounded Q&A, ingest, linting, and cited answer storage. |

## Example Prompts

```text
Find Samsung Electronics using list_companies.
```

```text
Use get_stock_prices for stock code 005930 from 2025-01-01 to 2025-03-31.
```

```text
Use list_disclosures with company_query "이노와이어리스" and show the latest 5 filings.
```

```text
Ask wiki_query about "삼성전기": what are the main products and supply chain risks?
```

## Architecture

Bullga MCP sits between MCP clients and the Bullga backend:

```text
MCP client
  -> Streamable HTTP /mcp
  -> FastMCP server
  -> Bullga Django REST API
  -> Bullga financial data stores and upstream market sources
```

The MCP layer keeps the interface agent-friendly: it normalizes company lookup, forwards requests to existing REST views, and returns structured JSON responses suitable for reasoning, analysis, and report generation.

## Notes

Bullga MCP provides data access tools for research and workflow automation. It does not provide investment advice. Validate data and assumptions before making financial decisions.
