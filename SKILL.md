---
name: rotifer-self-evolving-agent
description: Rank an Agent's Rotifer Genes against the Arena and swap in stronger ones. Invoked explicitly via /evolve — scan local capabilities, compare Genes, inspect fitness scores, and replace weak ones with user approval. Not for capabilities outside Rotifer.
license: Apache-2.0
compatibility: Requires network access and npx to run @rotifer/mcp-server@0.11.0 for Arena rankings and Gene metadata.
metadata:
  author: rotifer-protocol
  version: "2.3.5"
  command: /evolve
  mcp-package: "@rotifer/mcp-server@0.11.0"
---

# Rotifer Self Evolving Agent

Your Agent gets stronger by competing, not by configuring. Scan capabilities, benchmark against [Arena rankings](https://rotifer.dev), and upgrade to fitter alternatives — driven by objective performance data, not opinions.

## Runtime Integration

This skill is invoked with `/evolve` and uses the Rotifer MCP Server at runtime:

```bash
npx @rotifer/mcp-server@0.11.0
```

The MCP server provides Gene search, Arena rankings, fitness details, comparison, and local install operations.

## Quick Start

### Evolve Your Agent
Scan your Agent's current setup and get upgrade recommendations:
```bash
/evolve
```
This analyzes your local Agent configuration, identifies capabilities below Arena median, and recommends higher-performing replacements.

### Check Agent Status
View your Agent's capability dashboard:
```bash
/evolve status
```
Shows all installed capabilities, their fitness scores, and overall Agent health.

### Upgrade a Capability
Replace a capability with a stronger alternative:
```bash
/evolve upgrade <name>
```
Finds the top-ranked alternative in the same domain, shows you the swap, and installs it **only after you approve**. This is the one command that changes what is installed: it replaces a Gene under `~/.rotifer/` with third-party code from the marketplace, which changes what your Agent does at runtime. Every other `/evolve` command except `create-agent` is read-only.

> **Never pass `force: true` to `install_gene`.** Overwriting a Gene in place is
> the one action here that cannot be undone — nothing in the CLI or the MCP
> server can restore what it replaced. When a Gene of that name already exists,
> stop and tell the user what is installed, what would replace it, and that they
> must remove the existing one themselves to proceed. This restriction lifts
> once Rotifer can roll an upgrade back.

## Discovery & Comparison

### Discover Capabilities
Find capabilities by what you need, not by internal names:
```bash
/evolve discover web scraping
/evolve discover --domain code.format
/evolve discover --fidelity Native
```

### Compare Candidates
Side-by-side fitness comparison:
```bash
/evolve compare <id-1> <id-2>
```

### Arena Rankings
See top performers in any domain:
```bash
/evolve arena search.web
/evolve arena code.format
```

### Inspect Details
Full technical details for a specific capability:
```bash
/evolve inspect <id>
```

## Agent Management

### Create a New Agent
```bash
/evolve create-agent <name>
```

### Run Your Agent
```bash
/evolve run-agent <name>
```

## How it Works

Under the hood, Rotifer uses **Genes** — atomic, transferable AI capabilities that compete in an **Arena**. The fittest Genes (measured by the fitness function **F(g)**) rise to the top of the rankings automatically. Ranking is the automatic part; putting a Gene on your machine is not.

```text
F(g) = [S_r · ln(1 + C_util) · (1 + R_rob)] / [L · Resource_Cost]
```

No voting, no human preference — pure runtime performance metrics determine which capabilities win.

### Capability Types (Fidelity)

| Type | Description |
|------|-------------|
| **Native** | Pure WASM — fully sandboxed, highest security |
| **Hybrid** | WASM logic + controlled external calls |
| **Wrapped** | Thin envelope around an external API |

### What "Evolve" Actually Does

When you run `/evolve`, the assistant:

1. Lists your local Agent's installed Genes (`list_local_agents` + `list_local_genes`)
2. Checks each Gene's fitness against Arena rankings (`get_gene_detail` + `get_arena_rankings`)
3. Identifies Genes scoring below the domain median
4. Searches for stronger alternatives (`search_genes`)
5. Presents a ranked upgrade plan with fitness comparisons

No Gene is replaced without your confirmation.

## Security & Transparency

### Runtime dependency
This Skill runs [`@rotifer/mcp-server@0.11.0`](https://www.npmjs.com/package/@rotifer/mcp-server/v/0.11.0) via `npx` at runtime. The package is **fetched from npm on first use** and cached locally. This is a standard MCP Skill pattern but means you are trusting remote code — review the source before use.

`/evolve run-agent` is a second such path: it invokes the `rotifer` CLI, and when that is not on `PATH` it falls back to `npx -y @rotifer/playground`.

- **Source code**: [github.com/rotifer-protocol/rotifer-mcp-server](https://github.com/rotifer-protocol/rotifer-mcp-server)
- **Verify**: `npm view @rotifer/mcp-server@0.11.0 dist.integrity`

### Network requests
Gene, Arena and profile queries go to the Rotifer public API at `rotifer.dev` (hosted on Supabase). Beyond that the MCP server makes one version check per day against `registry.npmjs.org`, caching the answer in `~/.config/rotifer/update-check.json`; `npx` reaches the same registry when it fetches or refreshes a package.

**Usage telemetry.** The MCP server also reports every tool call to Rotifer Cloud, so this is worth being exact about. After each call it sends, fire-and-forget, the tool's name, the Gene id the call acted on (when there is one), whether it succeeded, its latency in milliseconds, and your Rotifer user id — `null` unless you are logged in. It is sent whether or not you log in, failures are ignored silently, and there is currently no way to turn it off. Running a Gene while logged in additionally records that invocation, as the protocol's anti-manipulation metrics depend on it.

What is **not** sent: the arguments you pass, the contents of any file, your environment variables, and your local configuration. Both loggers are `logMcpCall` and `logGeneInvocation` in [`src/cloud.ts`](https://github.com/rotifer-protocol/rotifer-mcp-server/blob/main/src/cloud.ts) — short enough to read in full, and a packet capture on first use will show you the same thing.

### Credentials
Public Gene and Arena data needs no login: the MCP server reads it with the **Supabase anon key** (a public, client-safe key protected by Row Level Security). It does not read your environment variables, nor secrets belonging to any other tool.

Logging in is explicit and optional — needed only to publish. `login` writes your Rotifer session token to `~/.rotifer/credentials.json` with `0600` permissions, and it is sent only to the Rotifer API. `logout` deletes it.

### Local data access
- **Reads** installed Genes under `~/.rotifer/`, and Agent definitions under `.rotifer/agents/` in the current project, to generate upgrade recommendations.
- Local configuration is **designed not to be transmitted** to any server. Comparison logic runs locally against data fetched from the public API. What does leave the machine on every call is the usage record described under [Network requests](#network-requests) — tool name, Gene id, outcome, latency, user id — and nothing beyond it. You can verify both by inspecting the [source code](https://github.com/rotifer-protocol/rotifer-mcp-server).
- **Writes** are confined to three places: Genes into `~/.rotifer/` (`install_gene`), Agent definitions into `.rotifer/agents/` in the current project (`create_agent`), and the update-check cache above. Nothing else on disk is modified, and no Gene is installed, replaced, or removed without explicit user confirmation.
- Removing this Skill does **not** roll back Genes it installed. They remain ordinary Genes under `~/.rotifer/`.

### Tool surface
The nine `/evolve` commands are a front-end over a much larger MCP tool set, which also covers compiling, publishing, Arena submission and login. This Skill does not invoke those, but installing it does place them within the assistant's reach — the full list is in the [server source](https://github.com/rotifer-protocol/rotifer-mcp-server).

### Permission justification
- `network:outbound` — required to query Arena rankings, Gene metadata, and fitness scores from the Rotifer public API, and to fetch the MCP server package itself from npm.

## Links

- [Creator Portal](https://rotifer.dev)
- [Capability Marketplace](https://rotifer.ai)
- [Documentation](https://rotifer.dev/docs)
- [Protocol Specification](https://github.com/rotifer-protocol/rotifer-spec)
