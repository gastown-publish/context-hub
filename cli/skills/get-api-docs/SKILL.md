---
name: get-api-docs
description: >
  Use this skill when you need documentation for a third-party library, SDK, or API
  before writing code that uses it — for example, "use the OpenAI API", "call the
  Stripe API", "use the Anthropic SDK", "query Pinecone", or any time the user asks
  you to write code against an external service and you need current API reference.
  Fetch the docs with gashub before answering, rather than relying on training knowledge.
---

# Get API Docs via gashub

When you need documentation for a library or API, fetch it with the `gashub` CLI
rather than guessing from training data. This gives you the current, correct API.

## Step 1 — Find the right doc ID

```bash
gashub search "<library name>" --json
```

Pick the best-matching `id` from the results (e.g. `openai/chat`, `anthropic/sdk`,
`stripe/api`). If nothing matches, try a broader term.

## Step 2 — Fetch the docs

```bash
gashub get <id> --lang py    # or --lang js, --lang ts
```

Omit `--lang` if the doc has only one language variant — it will be auto-selected.

## Step 3 — Use the docs

Read the fetched content and use it to write accurate code or answer the question.
Do not rely on memorized API shapes — use what the docs say.

## Step 4 — Annotate what you learned

After completing the task, if you discovered something not in the doc — a gotcha,
workaround, version quirk, or project-specific detail — save it so future sessions
start smarter:

```bash
gashub annotate <id> "Webhook verification requires raw body — do not parse before verifying"
```

Annotations are local, persist across sessions, and appear automatically on future
`gashub get` calls. Keep notes concise and actionable. Don't repeat what's already in
the doc.

## Step 5 — Give feedback

Always rate the doc after using it. This helps authors fix outdated or incorrect
docs and prioritize improvements. Include a label and a brief comment explaining
what was good or what needs fixing — specific feedback is the most valuable.

```bash
gashub feedback <id> up --label accurate "Clear examples, models are current"
gashub feedback <id> down --label outdated "Lists gpt-4o as latest but gpt-5.4 is out"
```

Available labels: `outdated`, `inaccurate`, `incomplete`, `wrong-examples`,
`wrong-version`, `poorly-structured`, `accurate`, `well-structured`, `helpful`,
`good-examples`.

If you notice the doc has wrong model names, deprecated APIs, missing features,
or incorrect code patterns, always leave a downvote with details so it can be fixed.

## Quick reference

| Goal | Command |
|------|---------|
| List everything | `gashub search` |
| Find a doc | `gashub search "stripe"` |
| Exact id detail | `gashub search stripe/api` |
| Fetch Python docs | `gashub get stripe/api --lang py` |
| Fetch JS docs | `gashub get openai/chat --lang js` |
| Save to file | `gashub get anthropic/sdk --lang py -o docs.md` |
| Fetch multiple | `gashub get openai/chat stripe/api --lang py` |
| Save a note | `gashub annotate stripe/api "needs raw body"` |
| List notes | `gashub annotate --list` |
| Rate a doc | `gashub feedback stripe/api up` |

## Notes

- `gashub search` with no query lists everything available
- IDs are `<author>/<name>` — confirm the ID from search before fetching
- If multiple languages exist and you don't pass `--lang`, gashub will tell you which are available
