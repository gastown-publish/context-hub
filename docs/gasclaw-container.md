# Gasclaw Container: gascontext

Design stub for a dedicated Gasclaw container that maintains `gastown-publish/context-hub`.

## Purpose

The `gascontext` container runs autonomous agents that:

1. Monitor upstream `andrewyng/context-hub` for updates and merge them
2. Validate content packs with `gashub build`
3. Smoke-test the MCP server after changes
4. Create PRs for content additions and fixes
5. Respond to Telegram messages in the context-hub topic

## Container Spec

| Setting | Value |
|---------|-------|
| Container name | `gascontext` |
| Image | `gasclaw` (shared base image) |
| Gateway port | 18797 |
| Telegram bot | `@gascontext_bot` (to be registered) |
| Telegram topic | TBD (in group `-1003810709807`) |
| Repo | `gastown-publish/context-hub` |
| Host bind-mount | `/home/gascontext/` |

## docker-compose Snippet

```yaml
services:
  gascontext:
    build:
      context: /home/nic/gasclaw-workspace/gasclaw
      platforms:
        - linux/amd64
    container_name: gascontext
    ports:
      - "18797:18797"
    volumes:
      - gascontext-openclaw:/root/.openclaw
      - gascontext-dolt:/root/.dolt
      - gascontext-state:/root/.gasclaw
      - gascontext-claude:/root/.claude-config
      - gascontext-workspace:/workspace
    env_file:
      - gascontext.env
    restart: unless-stopped

volumes:
  gascontext-openclaw:
  gascontext-dolt:
  gascontext-state:
  gascontext-claude:
  gascontext-workspace:
```

## Environment Variables

```bash
# gascontext.env
ANTHROPIC_BASE_URL=https://api.minimax.villamarket.ai
ANTHROPIC_API_KEY=sk-LITELLM_KEY
GASTOWN_KIMI_KEYS=sk-LITELLM_KEY
OPENCLAW_KIMI_KEY=sk-LITELLM_KEY
TELEGRAM_BOT_TOKEN=YOUR_BOT_TOKEN
TELEGRAM_OWNER_ID=2045995148
TELEGRAM_GROUP_IDS=-1003810709807
TELEGRAM_ALLOW_IDS=2045995148
GT_RIG_URL=https://TOKEN@github.com/gastown-publish/context-hub.git
GT_AGENT_COUNT=2
DOLT_PORT=3312
GATEWAY_PORT=18797
MONITOR_INTERVAL=300
ACTIVITY_DEADLINE=3600
```

## OpenClaw Config

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "moonshot/kimi-k2.5"
      }
    },
    "list": [
      {
        "id": "main",
        "name": "Context Hub Overseer",
        "emoji": "📚",
        "subagents": { "allowAgents": ["*"] }
      },
      {
        "id": "content-curator",
        "name": "Content Curator",
        "emoji": "📝",
        "subagents": { "allowAgents": ["*"] }
      },
      {
        "id": "mcp-tester",
        "name": "MCP Tester",
        "emoji": "🔌",
        "subagents": { "allowAgents": ["*"] }
      }
    ]
  },
  "channels": {
    "telegram": {
      "groups": {
        "-1003810709807": {
          "requireMention": true,
          "topics": {
            "TOPIC_ID": {
              "requireMention": false
            }
          }
        }
      }
    }
  }
}
```

## AGENTS.md (Workspace)

```markdown
# Agent Team — Context Hub

## main (📚 Context Hub Overseer)
Primary agent. Coordinates content updates, responds to Telegram, manages PRs.

## content-curator (📝 Content Curator)
Validates content packs, runs gashub build, checks quality, merges upstream.

## mcp-tester (🔌 MCP Tester)
Smoke-tests MCP server after changes, verifies tool responses.
```

## Phase 2 Implementation

Full production wiring (building the container, registering the bot, creating the Telegram topic, activating agents) is tracked in the [gasclaw-management](https://github.com/gastown-publish/gasclaw-management) repo under the Context Hub Epic.
