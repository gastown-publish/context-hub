# Context Hub — Gasclaw Fork

Curated, versioned docs and skills for AI coding agents. Agents search, fetch, annotate, and give feedback — building knowledge across sessions instead of hallucinating APIs.

This repo is a fork of [andrewyng/context-hub](https://github.com/andrewyng/context-hub) (MIT license), extended with an MCP server integration and managed by the [Gasclaw platform](https://github.com/gastown-publish/gasclaw-management).

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![npm](https://img.shields.io/npm/v/@gastown/chub)](https://www.npmjs.com/package/@gastown/chub)
[![Node.js](https://img.shields.io/badge/node-%3E%3D18-brightgreen)](https://nodejs.org)

## Quick Start

```bash
npm install -g @gastown/chub
chub search openai                 # find what's available
chub get openai/chat --lang py     # fetch current docs (Python version)
```

For the full CLI reference, see [docs/cli-reference.md](docs/cli-reference.md).

## How It Works

Chub is designed for your coding agent to use. Prompt your agent: *"Use the CLI command chub to get the latest API documentation. Run `chub help` to understand how it works."* Or install the [SKILL.md](cli/skills/get-api-docs/SKILL.md) into your agent's skills directory.

**Search, fetch, use:**

```bash
chub search "stripe payments"        # find relevant docs
chub get stripe/api --lang js        # fetch the doc
# Agent reads the doc, writes correct code.
```

**Annotate for next time:**

```bash
chub annotate stripe/api "Needs raw body for webhook verification"
# Next session, the annotation appears automatically on chub get.
```

**Feedback flows back to authors:**

```bash
chub feedback stripe/api up          # vote docs up or down
```

## MCP Server

Context Hub includes a built-in MCP (Model Context Protocol) server so agents can call `search` and `get` without shelling out to the CLI.

```bash
chub-mcp                            # starts MCP server on stdio
```

### Cursor

Add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "context-hub": {
      "command": "chub-mcp"
    }
  }
}
```

### Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "context-hub": {
      "command": "chub-mcp"
    }
  }
}
```

### OpenClaw

Add to `openclaw.json` tools section:

```json
{
  "tools": {
    "context-hub": {
      "type": "mcp",
      "command": "chub-mcp"
    }
  }
}
```

## Commands

| Command | Purpose |
|---------|---------|
| `chub search [query]` | Search docs and skills (no query = list all) |
| `chub get <id> [--lang py\|js]` | Fetch docs or skills by ID |
| `chub annotate <id> <note>` | Attach a note to a doc or skill |
| `chub annotate <id> --clear` | Remove annotations |
| `chub annotate --list` | List all annotations |
| `chub feedback <id> <up\|down>` | Upvote or downvote a doc |

## Tailscale / Private Registries

For internal-only content, use `path:` sources in `~/.chub/config.yaml`:

```yaml
sources:
  - type: path
    path: /home/nic/gasclaw-workspace/context-hub/content
  # Never commit API keys or Tailscale keys to this file
```

This avoids relying on public CDN and keeps content on your Tailscale network.

## Gasclaw Platform

This repo is maintained by the **Gasclaw multi-container AI agent platform** running on 8x NVIDIA H100 GPUs. The platform manages GitHub repos through Telegram bots and autonomous agents.

- **Management repo**: [gastown-publish/gasclaw-management](https://github.com/gastown-publish/gasclaw-management) (`/home/nic/gasclaw-workspace/gasclaw-management`)
- **Container**: `gascontext` (dedicated context-hub maintainer)
- **Telegram**: Bot in the `gastown_publish` group, topic-bound
- **Workflow**: Issues, PRs, CI, and releases run through Gasclaw agents

See [docs/gasclaw.md](docs/gasclaw.md) for the full management workflow.

## Self-Improving Agents

**Annotations** are local notes that persist across sessions — agents learn from past experience. **Feedback** (up/down ratings) goes to doc authors to improve content for everyone. See [Feedback and Annotations](docs/feedback-and-annotations.md).

## Contributing

Anyone can contribute docs and skills — API providers, framework authors, and the community. Content is plain markdown with YAML frontmatter, submitted as pull requests. See the [Content Guide](docs/content-guide.md).

For Gasclaw-managed contributions, see [docs/gasclaw.md](docs/gasclaw.md).

## Fork Relationship

This repo is a fork of [andrewyng/context-hub](https://github.com/andrewyng/context-hub). We extend it with:

- MCP server documentation for Gasclaw integration
- Gasclaw platform management workflow
- Dedicated container design for automated content maintenance
- Internal registry and Tailscale-only deployment patterns

Upstream changes can be pulled via `git fetch upstream && git merge upstream/main`.

## License

[MIT](LICENSE) — see [NOTICE](NOTICE) for fork attribution.
