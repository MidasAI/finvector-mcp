# FinVector MCP server

**SEC filing search for AI agents. Every answer cites the filing it came from.**

FinVector is a hosted [Model Context Protocol](https://modelcontextprotocol.io) server and REST API. Ask a question in plain
English and your agent gets back the passages from SEC filings, such as 10-Ks and 10-Qs, that answer it, best first, each with
its company, form, filing date, section, a link to the filing on sec.gov, and a link that opens the passage highlighted in the
FinVector app.

- **Cited, not summarised.** Your agent quotes the filing, and `verify_quote` checks a quote word for word against its source.
- **Broad coverage.** More than 5,000 US-listed companies and 1.6 million documents filed since late 2023, updated daily: 10-K, 10-Q, 8-K, proxies, registration statements, ownership filings and filings by foreign issuers. Ask `get_coverage` for the current list.
- **Search by meaning.** Hybrid keyword and semantic search with reranking, so "dependence on one supplier" finds "single-source components".
- **Screen by theme.** `screen_theme` lists which companies discuss a theme, quarter by quarter, with who is new and who dropped out.
- **One line to connect.** Works in Claude Code, Cursor, Claude Desktop, VS Code, Codex, Gemini CLI, Zed, Cline and any client that speaks Streamable HTTP.
- **Pay as you go.** 1 cent a search, 2 cents a screen. New accounts get 500 free credits.

```
MCP endpoint   https://api.finvector.ai/v1/mcp
REST API       https://api.finvector.ai/v1   (OpenAPI: https://api.finvector.ai/v1/openapi.json)
Docs           https://app.finvector.ai/api-docs
```

## Get started

1. Sign up at **[app.finvector.ai](https://app.finvector.ai)** and create an API key on the **Keys** page. It is shown once, so copy it.
2. Put the key in your environment: `export FINVECTOR_API_KEY=fv_prod_...`
3. Add FinVector to your tool (below), then try one of the [example prompts](#example-prompts).

The key goes in a header, `Authorization: Bearer <key>` or `X-API-Key: <key>`. Never put it in a URL: FinVector refuses a
key sent as a query parameter.

## Connect your tool

### Claude Code

```bash
claude mcp add --transport http finvector https://api.finvector.ai/v1/mcp --header "Authorization: Bearer $FINVECTOR_API_KEY"
```

Or share it with your team through a project `.mcp.json`, which reads each person's own key: [`examples/claude-code.mcp.json`](examples/claude-code.mcp.json).

### Cursor

Save as `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project): [`examples/cursor.mcp.json`](examples/cursor.mcp.json)

```json
{
  "mcpServers": {
    "finvector": {
      "url": "https://api.finvector.ai/v1/mcp",
      "headers": { "Authorization": "Bearer ${env:FINVECTOR_API_KEY}" }
    }
  }
}
```

### Claude Desktop

Claude Desktop connects through [mcp-remote](https://github.com/geelen/mcp-remote), a small bridge that runs on your computer.
Open **Settings > Developer > Edit Config** and add: [`examples/claude_desktop_config.json`](examples/claude_desktop_config.json)

```json
{
  "mcpServers": {
    "finvector": {
      "command": "npx",
      "args": ["mcp-remote", "https://api.finvector.ai/v1/mcp", "--transport", "http-only", "--header", "Authorization:${AUTH_HEADER}"],
      "env": { "AUTH_HEADER": "Bearer <your API key>" }
    }
  }
}
```

Restart Claude Desktop after saving. It needs Node.js for `npx`.

### VS Code (GitHub Copilot)

Run **MCP: Open User Configuration** and add: [`examples/vscode.mcp.json`](examples/vscode.mcp.json). VS Code asks for the key once and stores it securely.

### More clients

| Client | Config file | Example |
|---|---|---|
| OpenAI Codex CLI | `~/.codex/config.toml` | [`examples/codex.config.toml`](examples/codex.config.toml) |
| Gemini CLI | `~/.gemini/settings.json` | [`examples/gemini-settings.json`](examples/gemini-settings.json) (use `httpUrl`, not `url`) |
| Zed | `settings.json` | [`examples/zed-settings.json`](examples/zed-settings.json) |
| Cline | MCP settings | [`examples/cline_mcp_settings.json`](examples/cline_mcp_settings.json) (keep `"type": "streamableHttp"`) |
| Microsoft Copilot Studio | Add a tool > MCP | Authentication: API key, Type **Header**, header name `X-API-Key` |
| Any other client | | Streamable HTTP at `https://api.finvector.ai/v1/mcp` with an `Authorization: Bearer <key>` header |

Claude.ai and ChatGPT connect through sign-in (OAuth), which is coming soon.

Check that your key works:

```bash
curl -s https://api.finvector.ai/v1/search \
  -H "Authorization: Bearer $FINVECTOR_API_KEY" -H 'content-type: application/json' \
  -d '{"query":"tariff exposure","filters":{"date_from":"2025-01-01","date_to":"2026-12-31"},"limit":3}'
```

## Tools

| Tool | What it does | Cost |
|---|---|---|
| `search_filings` | Finds the passages that answer a question, best first, each with its company, form, filing date, section, a link to sec.gov (`url`) and a link that opens it in the FinVector app (`app_url`). | 1 credit |
| `screen_theme` | Which companies discuss a theme, most relevant first, quarter by quarter, with the names that are new and the ones that dropped out. | 2 credits |
| `verify_quote` | Checks a quote word for word against the section it cites, and shows the closest passage and every difference when it does not match. | Free |
| `get_filing_section` | Reads a whole section, such as a 10-K's risk factors, a page at a time. | Free |
| `get_filing` | A filing's company, form, dates and the sections it has, with its EDGAR link. | Free |
| `list_filings` | A company's most recent filings, newest first. | Free |
| `find_company` | Finds tickers and names from a company name or ticker, typos included. | Free |
| `get_coverage` | Which companies, forms and dates are in the index. | Free |

Every tool is read-only. Filing text is returned as untrusted third-party content, with invisible characters neutralised and
instruction-like text flagged, so an agent can treat it as data.

## Example prompts

- *What does Apple say about its dependence on single-source suppliers in its latest 10-K? Quote the passage and verify the quote against the filing.*
- *Which companies discussed tariff exposure in their 10-K risk factors over the past year? Show who is new each quarter.*
- *Compare how NVIDIA and AMD describe export controls in their most recent annual reports, with a citation for each point.*
- *Summarize the liquidity discussion in Microsoft's latest 10-Q, section by section, and link the filing.*

Use `get_coverage` (or ask "what companies does FinVector cover?") to see what is in the index.

## Pricing

1 USD = 100 credits. A search costs 1 credit, a theme screen 2; lookups, section reads and quote checks are free. Each new
verified account gets 500 free credits. When the balance runs out, calls return `402 usage_limit_reached`.

## Support and legal

- Docs: [app.finvector.ai/api-docs](https://app.finvector.ai/api-docs)
- Support: [app.finvector.ai/support](https://app.finvector.ai/support)
- [Privacy](https://app.finvector.ai/privacy) · [Terms](https://app.finvector.ai/terms)
- Issues with this guide: open an issue in this repository.

FinVector returns text from public SEC filings, with a link to each source. It is a research tool, not investment, legal or
tax advice. Check important facts against the filing itself before you rely on them.

This repository holds setup guides and configuration examples, under the [MIT license](LICENSE); the FinVector service itself is hosted. FinVector is built by [Midas](https://getmidas.ai).
