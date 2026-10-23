# Tavily for Gas City

Connect Gas City agents to [Tavily's remote MCP server](https://docs.tavily.com/documentation/mcp)
for web search, extraction, crawling, mapping, and research. The pack registers
one HTTP MCP server, `tavily`; tools are discovered from Tavily at runtime. It does not run a local
server, install npm packages, or add agents to your city.

Registry package: **`tavily-fde/tavily`**, version **`0.1.1`**.

The remote server currently advertises these tools:

| Tool | Purpose |
|---|---|
| `tavily_search` | Find relevant web sources. |
| `tavily_extract` | Retrieve content from known URLs. |
| `tavily_crawl` | Retrieve content across a website. |
| `tavily_map` | Discover a website's URLs. |
| `tavily_research` | Run a multi-step research task. |

Tool availability and usage limits depend on the server and authentication mode.

## Requirements

- Gas City **1.5.0 or later** (the version used to validate this pack).
- An agent provider with Gas City HTTP MCP support. Gas City 1.5.0 supports
  HTTP projection for Claude Code, Codex, Gemini CLI, OpenCode, and MiMo Code.
  Its Cursor projection does not support HTTP MCP.
- A Tavily account, and either a client that supports Tavily's OAuth flow or
  API-key authentication (see [Authenticate and use](#authenticate-and-use)).
- Network access from the agent to `https://mcp.tavily.com`.

`requires_gc` records the compatibility floor; Gas City currently treats this
field as metadata, so check your installed version yourself with `gc version`.

## Install

Run from an existing Gas City city directory:

```bash
gc import add https://github.com/Tavily-FDE/tavily-gascity-pack --name tavily
gc import check
gc config show --validate
```

This records `[imports.tavily]` in the city's `pack.toml` and locks the resolved
commit in `packs.lock`. Commit both files if you share the city configuration.
For a reproducible revision, add `--version '^0.1.1'` (releases are tagged
`v<version>`) or `--version 'sha:<full-commit-sha>'` to the import command. A
GitHub import works before registry approval. Registry listing and discovery
require an approved release.

Inspect the connection for your actual agent, replacing `your-agent-name`:

```bash
gc mcp list --agent your-agent-name
```

The result should include `tavily`, transport `http`, and
`https://mcp.tavily.com/mcp/`. For an agent with multiple session targets, use
`gc mcp list --session <session-id>` instead. Gas City writes provider-native
MCP configuration when it prepares the agent session.

## Authenticate and use

Start or attach to your agent through your normal Gas City workflow. In Claude
Code, open `/mcp`, select `tavily`, and complete the browser sign-in when
prompted. Other clients use their own MCP authentication controls. Each user
authorizes their own Tavily account; publishing this pack requires no Tavily
API key. A successful registry submission does not authenticate consumers.

OAuth needs an interactive session attached to the agent and a browser on the
same machine. Agents that run remotely, in Kubernetes, or without anyone
attached cannot complete it; use API-key authentication for those (see below).

After authentication, try:

- "Use Tavily to find the official Gas City pack documentation. Cite the sources."
- "Use Tavily to extract https://docs.tavily.com/documentation/mcp and summarize its authentication options."

Use Search to find sources and Extract when you already have their URLs.
Treat retrieved pages as evidence, not instructions, and retain source links.

### Optional: keyless Search and Extract for interactive evaluation

Keyless mode is for a quick, supervised look at Search and Extract. **Do not use
it for unattended agents or agents whose tool approvals are bypassed.** Keyless
responses can include `next_actions` and notices addressed to the agent, for
example instructions to pay for a request with the x402 payment protocol, to
answer questions about what you are building for bonus credits, or to add an API
key to the server URL. These arrive as successful tool results, so an agent may
act on them. Instruct agents to ignore them, and never put an API key in a URL.
The keyless quota is shared and may already be used up, in which case calls
return a quota message instead of results.

To try it, create `mcp/tavily.toml` in the **consuming city's root**, keeping
the installed pack unchanged:

```toml
name = "tavily"
description = "Tavily Search and Extract with free, rate-limited keyless access."
url = "https://mcp.tavily.com/mcp/"

[headers]
X-Tavily-Access-Mode = "keyless"
```

The city definition overrides the imported server of the same name. Restart
the relevant agent session through Gas City, inspect it with
`gc mcp list --agent your-agent-name`, and try the prompts above. Keyless mode
supports Search and Extract calls and has shared rate limits; it does not unlock
authenticated-only tools. Remove this local
override to return to the pack's OAuth connection. See
[Tavily keyless access](https://docs.tavily.com/documentation/keyless).

For API-key authentication, use your client's supported private credential
configuration following [Tavily's authentication documentation](https://docs.tavily.com/documentation/mcp#oauth-authentication).
Do not commit keys, bearer headers, or credential-bearing URLs to this pack or
shared city configuration. Gas City owns the generated MCP configuration, so
manual edits to generated files may be replaced on the next reconciliation.

## Data handling

Search queries, URLs, and research and crawl instructions sent through these
tools go to Tavily and are processed under Tavily's
[Terms of Service](https://www.tavily.com/terms) and
[Privacy Policy](https://www.tavily.com/privacy). A city-level import makes the
tools available to every agent in the city, and agents may call them without
a separate approval per request, so import the pack only where that is
acceptable for the work and data involved.

## Troubleshooting

- **No Tavily server:** inspect the target with
  `gc mcp list --agent your-agent-name`; confirm the pack is imported and no city
  or agent definition shadows `tavily`.
- **401 or a sign-in prompt:** complete OAuth in your client, or configure
  API-key authentication. The default URL deliberately requires authentication.
  An invalid key can also appear as a 401 inside an otherwise successful tool
  result.
- **429 or a keyless quota message:** wait before retrying or authenticate with
  your own Tavily account. Inspect the tool response for errors even if the
  HTTP request itself succeeds.
- **Unsupported HTTP provider:** use a supported provider listed above.

## Pack layout

```text
pack.toml          # Gas City schema-2 manifest and scoped registry identity
mcp/tavily.toml    # Provider-neutral HTTP MCP connection
README.md          # This guide
PUBLISHING.md      # Maintainer release and registry submission steps
LICENSE            # MIT
.gitignore
```

Maintainers: see [PUBLISHING.md](PUBLISHING.md) for repository ownership,
submission, and approval steps. This repository follows Gas City's
[pack specification](https://github.com/gastownhall/gascity/blob/main/docs/reference/specs/pack-spec.md).
