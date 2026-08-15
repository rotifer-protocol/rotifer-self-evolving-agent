# Rotifer Self Evolving Agent

> Your Agent gets stronger by competing, not by configuring. Scan capabilities, benchmark against Arena rankings, and upgrade to whatever wins.

Ranking and selection are automatic — driven by measured performance, not
opinions. **Installing is not.** Nothing on your machine changes until you say
so; see [What this Skill changes on your machine](#what-this-skill-changes-on-your-machine).

## Installation

Install from ClawHub:
1. Open OpenClaw
2. Search for "rotifer-self-evolving-agent" in the Skill marketplace
3. Click "Install"

Or manually:
```bash
cp -r rotifer-self-evolving-agent/ ~/.openclaw/workspace/skills/rotifer-self-evolving-agent/
```

## Usage

```bash
/evolve                          # Scan Agent, recommend upgrades
/evolve status                   # Agent capability dashboard
/evolve upgrade <name>           # Replace with stronger alternative
/evolve discover <query>         # Find capabilities by need
/evolve arena <domain>           # View Arena rankings
/evolve compare <id1> <id2>      # Compare candidates
/evolve inspect <id>             # Full capability details
/evolve create-agent <name>      # Compose an Agent from local Genes
/evolve run-agent <name>         # Run a local Agent
```

## How it Works

This Skill wraps the [Rotifer MCP Server](https://www.npmjs.com/package/@rotifer/mcp-server), which connects to the [Rotifer Protocol](https://rotifer.dev) — an evolution framework where AI capabilities (Genes) compete in an Arena. The fittest survive based on objective runtime metrics via the F(g) fitness function.

Key MCP tools used:

| Command | MCP Tools |
|---------|-----------|
| `evolve` | `list_local_agents` → `get_gene_detail` → `get_arena_rankings` → `search_genes` |
| `status` | `list_local_agents` + `list_local_genes` + `get_gene_detail` |
| `upgrade` | `get_arena_rankings` → `compare_genes` → `install_gene` |
| `discover` | `search_genes` |
| `inspect` | `get_gene_detail` |
| `compare` | `compare_genes` |
| `arena` | `get_arena_rankings` |

## What this Skill changes on your machine

This Skill exists to modify your Agent's installed capabilities, so it is worth
being precise about what that means. The full version is in
[SKILL.md](SKILL.md#security--transparency).

| | |
|---|---|
| **Runs remote code** | `npx @rotifer/mcp-server@0.11.0`, fetched from npm on first use and cached. `/evolve run-agent` additionally shells out to the `rotifer` CLI, falling back to `npx -y @rotifer/playground` when it is not installed. Both are remote code — [read the source](https://github.com/rotifer-protocol/rotifer-mcp-server) or check `npm view @rotifer/mcp-server@0.11.0 dist.integrity` first. |
| **Reads** | Installed Genes under `~/.rotifer/`, and Agents under `.rotifer/agents/` in the current project. |
| **Writes** | Genes into `~/.rotifer/` (`/evolve upgrade`); Agent definitions into `.rotifer/agents/` in the current project (`/evolve create-agent`); an update-check cache at `~/.config/rotifer/update-check.json`. Nothing outside those three. |
| **Sends** | Gene and Arena queries to the public Rotifer API, and one version check per day to `registry.npmjs.org`. Your local configuration is not uploaded — ranking is compared locally against data fetched from the API. |
| **Credentials** | It does not read your environment variables or other tools' secrets. Public data needs no login. If you do log in, your Rotifer session token is written to `~/.rotifer/credentials.json` with `0600` permissions and sent only to the Rotifer API. |

**Every replacement is confirmed by you.** `/evolve` and `/evolve status` only
read and report. `/evolve upgrade` proposes a specific swap and waits — no Gene
is installed, replaced, or removed without an explicit yes.

Two things worth knowing beyond that:

- The MCP server behind this Skill exposes far more tools than the nine
  `/evolve` commands — publishing, compiling and login among them. The Skill
  does not use them, but installing it does put them within the assistant's
  reach.
- Removing the Skill does not roll back capabilities it installed. They are
  ordinary Genes under `~/.rotifer/`; remove them as you would any other.

## Links

- [Rotifer Protocol](https://rotifer.dev)
- [Capability Marketplace](https://rotifer.ai)
- [Documentation](https://rotifer.dev/docs)
- [MCP Server](https://www.npmjs.com/package/@rotifer/mcp-server)

## License

Apache-2.0
