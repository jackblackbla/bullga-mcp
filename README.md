# Bullga MCP

AI-native access to Korean financial market data through a hosted Model Context Protocol server.

- **Endpoint:** `https://mcp.bullga.ai/mcp`
- **Project:** [Bullga.ai](https://bullga.ai)
- **Docs:** [Tools](docs/tools.md) · [Access Control](docs/access-control.md) · [Agent QA](docs/qa.md) · [Hosted Overview](https://docs.bullga.ai/docs/intro)

## Connect

No local server is required. Add the hosted Streamable HTTP endpoint to your MCP client.

### Codex

```bash
codex mcp add bullga --url https://mcp.bullga.ai/mcp
codex mcp list
```

### Claude Code

```bash
claude mcp add --transport http bullga https://mcp.bullga.ai/mcp
```

Run `/mcp` inside Claude Code to confirm Bullga is connected.

### Claude Desktop

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

## Authentication

Anonymous callers can use read tools within Free policy limits.

For authenticated workflows, send a personal MCP API key as either header:

```http
Authorization: Bearer bg_mcp_xxx
```

```http
X-API-Key: bg_mcp_xxx
```

Authenticated access is required for user-scoped tools such as watchlists. Pro-or-higher access is required for wiki write/admin tools.

See [Access Control](docs/access-control.md) for quotas, credit weights, and error semantics.

## What You Can Ask For

Bullga MCP exposes 44 tools across company, stock, disclosure, financial-statement, ownership, trading, theme, news, watchlist, and wiki data.

Example prompts:

```text
Use get_ownership for 부산산업 and summarize the largest shareholder and related-party ownership.
```

```text
Use get_stock_price_chart, list_news_articles, and list_disclosures to explain whether 삼성전자가 최근 일주일간 오른 이유를 근거 있게 말할 수 있는지 확인해줘.
```

```text
Use get_financial_statement_values for 005930 and 000660 with columns 매출액, 영업이익, 자산, 부채.
```

## Tool Reference

- [Tool Surface](docs/tools.md) — full tool list, access level, and credit weight.
- [Agent QA](docs/qa.md) — fixed Korean finance cases used to check agent tool selection.
- [Access Control](docs/access-control.md) — anonymous, API-key, JWT-session, and plan behavior.

## Notes

Bullga MCP is for research and workflow automation. It is not a real-time trading feed and does not provide investment advice. Validate data and assumptions before making financial decisions.
