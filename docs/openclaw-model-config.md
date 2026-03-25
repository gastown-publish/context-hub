# OpenClaw Model Configuration — Lessons Learned

## The Problem

After gateway restarts, Docker rebuilds, or config changes, agents fail with:
```
Unknown model: moonshot/minimax-m2.5 (model_not_found)
```
or
```
HTTP 401 authentication_error: kimi-coding/k2p5
```

## Root Cause

OpenClaw has **three** places where the model is configured, and they must all agree:

1. **`openclaw.json`** → `agents.defaults.model.primary` (e.g. `moonshot/kimi-k2.5`)
2. **`agents/*/agent/models.json`** → `providers.moonshot.models[].id` (e.g. `kimi-k2.5`)
3. **`agents/*/agent/models.json`** → `providers.moonshot.baseUrl` (must point to LiteLLM)

The format is `<provider>/<model-id>`. The provider name (`moonshot`) must match a key in `models.json.providers`, and the model-id (`kimi-k2.5`) must match a `models[].id` entry under that provider.

### What Goes Wrong

- **Gateway restart** reloads models from `models.json`. If the file was overwritten with defaults (e.g. `minimax-m2.5` instead of `kimi-k2.5`), the gateway won't find the model.
- **Docker container restart** may restore default `models.json` from the image, reverting `baseUrl` to `api.moonshot.ai` (upstream) instead of `api.minimax.villamarket.ai` (our LiteLLM).
- **Duplicate gateway processes** cause `409: Conflict: terminated by other getUpdates request` on Telegram because two instances poll the same bot token.
- **The `apply_minimax_fix_all_gasclaw.sh` script** changes `openclaw.json` to `moonshot/minimax-m2.5` but doesn't update `models.json` to match, causing a mismatch.

## The Fix

### Working Configuration

**`openclaw.json`**:
```json
{
  "agents": {
    "defaults": {
      "model": {"primary": "moonshot/kimi-k2.5"}
    }
  }
}
```

**`agents/*/agent/models.json`** (MUST be the same in ALL agent dirs):
```json
{
  "providers": {
    "moonshot": {
      "baseUrl": "https://api.minimax.villamarket.ai/v1",
      "api": "openai-completions",
      "models": [
        {"id": "kimi-k2.5", "name": "Kimi K2.5", "reasoning": false,
         "input": ["text","image"],
         "cost": {"input":0,"output":0,"cacheRead":0,"cacheWrite":0},
         "contextWindow": 262144, "maxTokens": 262144}
      ],
      "apiKey": "MOONSHOT_API_KEY"
    }
  }
}
```

### Key Rules

1. **`primary` in openclaw.json** = `moonshot/kimi-k2.5` (provider/model-id)
2. **`models.json` model id** = `kimi-k2.5` (matches the part after `/`)
3. **`baseUrl`** = `https://api.minimax.villamarket.ai/v1` (our LiteLLM, NOT api.moonshot.ai)
4. **Copy working `models.json`** from `gasclaw-minimax` to all other containers after any restart
5. **Never have duplicate gateway processes** — kill all node processes before starting a new gateway

### Recovery Script

```bash
# Copy working config from minimax to all containers
for c in gasclaw-dev gasclaw-gasskill gasclaw-context gasclaw-mgmt; do
  docker cp gasclaw-minimax:/root/.openclaw/agents/main/agent/models.json /tmp/models.json
  docker exec $c bash -c "
    for d in /root/.openclaw/agents/*/agent/; do
      cat > \$d/models.json" < /tmp/models.json
    done
  "
done

# Restart gateways (kill old first!)
for entry in "gasclaw-dev:18794" "gasclaw-gasskill:18796" "gasclaw-context:18797" "gasclaw-mgmt:18798"; do
  c="${entry%%:*}"; p="${entry##*:}"
  docker exec $c bash -c "
    for f in /proc/[0-9]*/exe; do
      target=\$(readlink \$f 2>/dev/null)
      echo \$target | grep -q node && kill -9 \$(echo \$f | cut -d/ -f3) 2>/dev/null
    done
    rm -f /root/.openclaw/gateway.lock
    sleep 1
    nohup openclaw gateway run --port $p --allow-unconfigured > /tmp/openclaw-gw.log 2>&1 &
  "
done
```

## LiteLLM Model Names

LiteLLM serves these models (all route to the same MiniMax-M2.5 on vLLM port 8080):

| Model ID | Works |
|----------|-------|
| `minimax-m2.5` | Yes (via LiteLLM) |
| `kimi-k2.5` | Yes (via LiteLLM, alias) |
| `MiniMax-M2.5` | Yes (via LiteLLM) |

In OpenClaw, use `moonshot/kimi-k2.5` because that's what the working `models.json` defines.
