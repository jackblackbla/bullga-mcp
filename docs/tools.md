# Bullga MCP Tool Surface

Last synced: **2026-05-23** from FastMCP registration in `jackblackbla/joah`.

Bullga MCP currently exposes **44 registered tools across 15 domains**. This list intentionally includes only tools registered into the hosted MCP surface; internal helpers and unregistered implementation functions are omitted.

| Domain | Registered tools and access | Notes |
| --- | --- | --- |
| Auth | `authenticate` — auth utility<br>`refresh_access_token` — auth utility<br>`logout` — auth utility | JWT session setup and teardown for session-authenticated workflows. |
| Companies | `list_companies` — standard read, 1 credit<br>`get_company` — standard read, 1 credit | Company search and canonical company identity lookup. |
| Convertible Bonds | `list_convertible_bonds` — standard read, 1 credit | Convertible bond issuance, conversion terms, and dilution context. |
| Disclosures | `list_disclosures` — standard read, 1 credit<br>`get_disclosure` — weighted read, 3 credits | DART disclosure search and full disclosure body retrieval. |
| Financial Statements | `list_financial_statements` — weighted read, 2 credits<br>`get_financial_statement` — weighted read, 2 credits<br>`get_financial_statement_values` — weighted read, 2 credits<br>`get_account_taxonomy_tree` — weighted read, 2 credits | Statement listings, detailed statements, account taxonomy, and spreadsheet-style value extraction. |
| Investor Trading | `list_investor_trading` — standard read, 1 credit | KRX investor-type trading activity by stock/date. |
| Market Indices | `list_market_indices` — standard read, 1 credit | Korean market index OHLCV, value, and daily change records. |
| News | `list_news_articles` — weighted read, 2 credits | Company-linked news listings for catalyst research. |
| Ownership | `get_ownership` — standard read, 1 credit | Major shareholders, related-party stakes, treasury shares, and voting-power estimates. |
| Short Selling | `list_short_selling_trades` — standard read, 1 credit<br>`list_short_selling_balances` — standard read, 1 credit | KRX short-selling trades and balance records. |
| Stock Wiki | `list_stock_wiki_index` — standard read, 1 credit<br>`get_stock_wiki_detail` — standard read, 1 credit | Bullga stock wiki index/detail pages for business and value-chain evidence. |
| Stocks | `list_stocks` — standard read, 1 credit<br>`get_stock_casual_info` — standard read, 1 credit<br>`get_stock_prices` — standard read, 1 credit<br>`get_stock_price_chart` — standard read, 1 credit<br>`get_stock_sparkline` — standard read, 1 credit<br>`get_stock_dividends` — standard read, 1 credit<br>`get_stock_derived_indicators` — weighted read, 2 credits | Stock screening, casual stock info, price history, chart/sparkline data, dividends, and derived indicators. |
| Themes | `list_themes` — weighted read, 2 credits<br>`get_theme` — weighted read, 2 credits<br>`get_theme_members` — weighted read, 3 credits<br>`get_theme_state_history` — weighted read, 3 credits<br>`get_theme_index_history` — weighted read, 3 credits | Public investment themes, members, confidence, index history, and state history. |
| Watchlists | `list_watchlists` — authenticated session or API key<br>`create_watchlist` — authenticated session or API key<br>`get_watchlist` — authenticated session or API key<br>`update_watchlist` — authenticated session or API key<br>`delete_watchlist` — authenticated session or API key<br>`list_watchlist_items` — authenticated session or API key<br>`add_watchlist_item` — authenticated session or API key<br>`remove_watchlist_item` — authenticated session or API key | Authenticated user watchlist management. |
| Wiki V2 | `wiki_get_page` — standard read, 1 credit<br>`wiki_ingest` — Pro+ write/admin, 5 credits<br>`wiki_lint` — standard read, 1 credit<br>`wiki_file_answer` — Pro+ write/admin, 5 credits | File-backed wiki page reads, ingest/lint operations, and cited answer filing. |

## Access Legend

- **auth utility** — session setup/refresh/logout; not metered by the normal access wrapper.
- **standard read, 1 credit** — available to anonymous, Free, Pro, and Enterprise callers within quota.
- **weighted read** — available within quota, but consumes the listed monthly-credit weight.
- **authenticated session or API key** — requires a Bullga user identity because the data is user-scoped.
- **Pro+ write/admin** — requires an authenticated Pro-or-higher user and consumes the listed credits.

## Company Resolution

Many tools accept `company_query` in addition to identifiers such as `corp_code`, `stock_code`, `code`, or `company_id`. Resolution can return:

- `AMBIGUOUS_COMPANY` with candidate companies.
- `COMPANY_NOT_FOUND` when no match exists.
- `MISSING_COMPANY_IDENTIFIER` when no usable identifier was supplied.

Agents should show candidates or ask the user to disambiguate instead of guessing.

## Recent Behavior Notes

- `get_ownership` accepts `company_query`, `corp_code`, `stock_code`/`code`, or `company_id` and resolves to the representative stock before querying ownership.
- `list_news_articles(company_query=...)` requires explicit company mention in article title, description, or body text.
- `get_disclosure` returns full body fields when available and can trigger server-side body hydration if they are missing.
- `get_financial_statement_values` is the narrow tool for spreadsheet-style extraction across multiple stock codes and account columns.
- `get_stock_casual_info` is a helper for identity/basic stock context; it is not a substitute for domain evidence tools such as `get_ownership`, `get_stock_derived_indicators`, or `get_stock_wiki_detail`.
