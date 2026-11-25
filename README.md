# Leadsets MCP server

[![Leadsets MCP connector](https://glama.ai/mcp/connectors/ai.leadsets.mcp/lead-sets/badges/score.svg)](https://glama.ai/mcp/connectors/ai.leadsets.mcp/lead-sets)

Leadsets is a remote [MCP](https://modelcontextprotocol.io) server that gives AI agents B2B data: people and company search, enrichment, email verification and company website scraping. Add one URL to your client and your agent gets eight tools. Nothing to install or run locally.

```
https://mcp.leadsets.ai
```

This repository holds connection guides and examples. The server itself is hosted by Leadsets; there is no code to run here. Full docs are at [leadsets.ai/docs/mcp](https://www.leadsets.ai/docs/mcp?utm_source=github&utm_medium=readme&utm_campaign=leadsets-mcp&utm_content=intro).

## Contents

- [Get an API key](#get-an-api-key)
- [Connect your client](#connect-your-client)
- [Tools](#tools)
- [Limits and pricing](#limits-and-pricing)
- [Check it works](#check-it-works)
- [Use the REST API or an SDK instead](#use-the-rest-api-or-an-sdk-instead)
- [Support and security](#support-and-security)

## Get an API key

Sign-up is waitlist-only until the public launch on 10 December 2026. [Join the waitlist](https://www.leadsets.ai/signup?utm_source=github&utm_medium=readme&utm_campaign=leadsets-mcp&utm_content=api-key). At launch, the free plan will include 1,000 enrichment records and 100 search records, no card. After you have an account, copy your key from **API keys** in the dashboard. Keys start with `ls_live_`. Keep yours private: don't commit it, paste it into a shared chat or put it in browser code.

## Connect your client

There are two ways to authenticate:

- **Sign in (OAuth):** clients such as Claude and ChatGPT open a Leadsets page where you paste your API key once and click **Allow**. The client then holds a token, not your key.
- **API key header:** clients such as Cursor and Claude Code send `Authorization: Bearer ls_live_…` with every request.

Either way, the connection has the same plan limits and access level as the key. Revoking the key in the dashboard disconnects every client that uses it.

### Claude (web and desktop)

1. Open **Customize**, then **Connectors**.
2. Click **+ Add**, then **Add custom connector**.
3. Name it `Leadsets` and paste the server URL: `https://mcp.leadsets.ai`
4. Click **Continue**, then sign in: paste your Leadsets API key on the Leadsets page and click **Allow**.
5. In a chat, click **+**, then **Connectors**, and turn Leadsets on.

On Team and Enterprise plans, an owner adds it once under **Organization settings, Connectors**.

### Claude Code

```bash
claude mcp add --transport http leadsets https://mcp.leadsets.ai \
  --header "Authorization: Bearer $LEADSETS_KEY"
```

Then start Claude Code and run `/mcp`. You'll see `leadsets` with 8 tools.

### Cursor

Open **Cursor Settings, MCP**, or edit `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "leadsets": {
      "url": "https://mcp.leadsets.ai",
      "headers": { "Authorization": "Bearer ls_live_…" }
    }
  }
}
```

Restart Cursor. You'll see **leadsets · 8 tools** with a green dot. The same file is in [`examples/cursor-mcp.json`](examples/cursor-mcp.json).

### ChatGPT

Custom MCP servers are available on ChatGPT Pro (read-only) and on Business, Enterprise and Edu. On Business, an admin or owner creates it.

1. Open **Plugins** in ChatGPT on the web.
2. Click **+**, then **Add custom MCP server**.
3. Name it `Leadsets` and paste `https://mcp.leadsets.ai` under **Connection, Server URL**.
4. Choose **OAuth** for authentication.
5. Confirm the warning, then **Create** it as a plugin.
6. Install it, then in a new chat type `@` and pick Leadsets.

### Other clients

Any client that supports remote MCP servers over streamable HTTP works:

1. Add a remote (HTTP) server with the URL `https://mcp.leadsets.ai`.
2. Sign in if the client offers OAuth. Otherwise add the header `Authorization: Bearer ls_live_…`.

A generic config is in [`examples/mcp.json`](examples/mcp.json).

Step-by-step guides for more clients are on the [integrations page](https://www.leadsets.ai/integrations?utm_source=github&utm_medium=readme&utm_campaign=leadsets-mcp&utm_content=integrations).

## Tools

| Tool | What it does | What it uses |
|---|---|---|
| `search_people` | Find people by title, seniority, function, company, company size, industry and country. Each result says why it matched. | 1 search record per record returned (person, company); 1 enrichment record per email |
| `search_companies` | Find companies by industry, headcount, country and technology. | 1 search record per record returned |
| `enrich_person` | A person from a LinkedIn URL, an email, or a name and company domain, with their work email when found. | 1 enrichment record when found |
| `enrich_company` | A company profile from a domain or LinkedIn URL. | 1 enrichment record when found |
| `verify_email` | Verify one email or up to 100: verified, catch-all, unknown or invalid, with a confidence score. | 1 enrichment record per email checked |
| `scrape_company` | Read a company's website and return structured content. | 1 scrape unit per page |
| `scrape_jobs` | Open roles from a company's careers pages. | 1 scrape unit per page |
| `get_usage` | What you've used and what's left on every limit. | Nothing |

In people search, each person is one record and their company another: pass `include_company: false` for people only. Work emails come with `include_emails: true` (or `has_verified_email`) and use 1 enrichment record each. Empty results use no records. Every record carries when it was last checked and a confidence score, and an email is only labelled verified when it is. Tool details are in the [API reference](https://www.leadsets.ai/docs/api/people-search?utm_source=github&utm_medium=readme&utm_campaign=leadsets-mcp&utm_content=tools).

## Limits and pricing

- **Free (at public launch on 10 December 2026):** 1,000 enrichment records and 100 search records (up to 10 per call), one time, no card. Scraping is not on Free. Until then, [join the waitlist](https://www.leadsets.ai/signup?utm_source=github&utm_medium=readme&utm_campaign=leadsets-mcp&utm_content=pricing).
- **Paid plans:** flat monthly prices with no credits, from $29 a month, with fair-use limits per rolling five hours and per week.
- **Scraping:** an add-on, sold in scrape units.

Every response says what the call used and what's left. Full numbers are on the [pricing page](https://www.leadsets.ai/pricing?utm_source=github&utm_medium=readme&utm_campaign=leadsets-mcp&utm_content=pricing) and in [limits and costs](https://www.leadsets.ai/docs/limits?utm_source=github&utm_medium=readme&utm_campaign=leadsets-mcp&utm_content=limits).

## Check it works

Ask your agent:

> What's my Leadsets usage?

It should call `get_usage` and answer with what you have left. If it does, you're connected.

Then try a real request:

> Find 10 VPs of Sales at US SaaS companies with 50 to 200 people, with verified emails.

## Use the REST API or an SDK instead

The same tools are available as a REST API at `https://api.leadsets.ai/v1`, with official SDKs:

- TypeScript and JavaScript: [`@leadsetsai/sdk`](https://github.com/leadsets/sdk) (`npm install @leadsetsai/sdk`)
- Python: [`leadsets`](https://github.com/leadsets/sdk-python) (`pip install leadsets`)

Ready-made agent skills that use these tools are in [leadsets-skills](https://github.com/leadsets/leadsets-skills).

## Support and security

- Questions: [support@leadsets.ai](mailto:support@leadsets.ai) or the [support page](https://www.leadsets.ai/support?utm_source=github&utm_medium=readme&utm_campaign=leadsets-mcp&utm_content=support).
- Security reports: see [SECURITY.md](SECURITY.md).

The guides and examples in this repository are MIT licensed. Use of the Leadsets service is covered by the [terms of service](https://www.leadsets.ai/terms?utm_source=github&utm_medium=readme&utm_campaign=leadsets-mcp&utm_content=terms).
