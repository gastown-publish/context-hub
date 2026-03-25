# Gasclaw Management Workflow

This repo (`gastown-publish/context-hub`) is maintained by the **Gasclaw platform** — a multi-container AI agent system that manages GitHub repos through Telegram bots and autonomous agents.

## Container Assignment

This repo is managed by the **`gascontext`** container (or `gasclaw-mgmt` as a secondary workspace). The dedicated container runs an OpenClaw gateway with agents specialized for content curation and MCP server maintenance.

| What | Value |
|------|-------|
| Container name | `gascontext` |
| Gateway port | 18797 |
| Telegram bot | TBD (to be registered) |
| Telegram topic | TBD (in group `-1003810709807`) |
| Repo | `gastown-publish/context-hub` |

## Absolute Paths (GPU Host)

| Resource | Path |
|----------|------|
| This repo (local clone) | `/home/nic/gasclaw-workspace/context-hub` |
| Platform management repo | `/home/nic/gasclaw-workspace/gasclaw-management` |
| Handoff prompt | `/home/nic/gasclaw-workspace/gasclaw-management/prompts/context-hub-fork-handoff.md` |
| Container user home | `/home/gascontext/` (if dedicated container) |
| Container workspace | `/workspace/gt` (inside container) |

## Bot and Telegram Workflow

Agents communicate via the `gastown_publish` Telegram forum group (`-1003810709807`):

- Each container's bot auto-responds in its own topic (no `@mention` needed)
- In General or other topics, `@mention` the bot explicitly
- Maintainers can trigger work by messaging the bot in its topic

### Agent Team

| Agent | Role |
|-------|------|
| `main` | Primary overseer — coordinates work, responds to messages |
| `content-curator` | Validates content packs, runs `gashubbuild`, checks quality |
| `mcp-tester` | Smoke-tests MCP server, verifies tool responses |

## CI and PR Workflow

- **CI must pass** before any merge — verify with `gh pr checks`
- Agents create branches, open PRs, and request review
- Beads tracks issues: `bd ready` (from gasclaw-management) shows work items
- Cross-repo issues filed in `gastown-publish/gasclaw-management` for platform-wide concerns

## Day-to-Day Operations

### Content Validation

```bash
# Inside container or from host
gashub build content/ -o dist/
gashub search "test query"    # verify index
gashub get <id> --lang py     # verify fetch
```

### MCP Smoke Test

```bash
gashub-mcp &                   # start MCP server
# Send tools/list via stdio, verify gashub_search and gashub_get appear
```

### Syncing Upstream

```bash
git fetch upstream
git merge upstream/main
# Resolve conflicts, run tests, push
```

## Cross-Links

- **Platform management**: [gastown-publish/gasclaw-management](https://github.com/gastown-publish/gasclaw-management)
- **Container design**: [docs/gasclaw-container.md](gasclaw-container.md)
- **Upstream**: [andrewyng/context-hub](https://github.com/andrewyng/context-hub)
- **HANDOFF.md**: Platform-wide handoff at `/home/nic/gasclaw-workspace/gasclaw-management/HANDOFF.md`
