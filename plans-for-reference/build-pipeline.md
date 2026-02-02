# Plan: Content Build Pipeline (`chub build`)

## Overview

A build command that scans a content repo, discovers DOC.md/SKILL.md files, and generates a `registry.json` suitable for serving as a chub source (CDN or local).

## New registry format

Top-level split into `docs[]` and `skills[]`. Skills have no language/version nesting.

```json
{
  "version": "1.0.0",
  "base_url": "https://cdn.contexthub.dev/v1",
  "generated": "2026-02-02T00:00:00.000Z",
  "docs": [
    {
      "name": "openai-chat",
      "description": "OpenAI Chat API - completions, streaming, function calling",
      "source": "maintainer",
      "tags": ["openai", "chat", "llm"],
      "languages": [
        {
          "language": "python",
          "versions": [
            {
              "version": "1.52.0",
              "path": "openai/docs/chat/v1",
              "files": ["DOC.md", "references/streaming.md"],
              "size": 42000,
              "lastUpdated": "2025-11-12"
            }
          ],
          "recommendedVersion": "1.52.0"
        },
        {
          "language": "javascript",
          "versions": [
            {
              "version": "1.52.0",
              "path": "openai/docs/chat/v1",
              "files": ["DOC.md", "references/streaming.md"],
              "size": 42000,
              "lastUpdated": "2025-11-12"
            }
          ],
          "recommendedVersion": "1.52.0"
        }
      ]
    }
  ],
  "skills": [
    {
      "name": "playwright-login",
      "description": "Login flow automation patterns for Playwright",
      "source": "community",
      "tags": ["browser", "playwright", "automation"],
      "path": "playwright-community/skills/login-flows",
      "files": ["SKILL.md", "helpers/login-util.ts"],
      "size": 12000,
      "lastUpdated": "2026-01-15"
    }
  ]
}
```

**Key changes from previous format:**
- `entries[]` → split into `docs[]` and `skills[]`
- `docs[]`: has `languages[].versions[]` nesting (language + version matter)
- `skills[]`: flat — just `name`, `path`, `files`, `size`, `lastUpdated` (no language/version)
- No more `provides` field — array membership IS the type
- `--lang` and `--version` flags only apply to `chub get docs`
- Bundled entries (same name has DOC.md + SKILL.md) → separate items in `docs[]` and `skills[]`

## Content repo structure

```
content-repo/
├── stripe/                              # author directory
│   ├── registry.json                    # OPTIONAL: author manages own index
│   ├── docs/
│   │   └── payments/
│   │       ├── DOC.md
│   │       └── references/
│   │           └── webhooks.md
│   └── skills/
│       └── integration/
│           ├── SKILL.md
│           └── scripts/
│               └── setup.sh
├── openai/                              # no registry.json → auto-discover
│   └── docs/
│       └── chat/
│           ├── DOC.md                   # languages: "python,javascript", versions: "1.52.0"
│           └── references/
│               └── streaming.md
└── playwright-community/
    └── skills/
        └── login-flows/
            ├── SKILL.md
            └── helpers/
                └── login-util.ts
```

**Convention**: `<author>/{docs,skills}/<entry-name>/` with DOC.md or SKILL.md at root.

## The entry directory is the unit of content

Following the convention established by Anthropic and OpenAI skill repos:
- SKILL.md (or DOC.md) lives in a directory
- All other files in that directory are companions — installed together with `--full`
- References use **relative paths** (e.g., `[Auth](references/auth.md)`)
- `--full -o <dir>` writes individual files preserving structure, so relative links resolve

## Two discovery modes per author directory

### 1. Author provides `registry.json`
Same schema as top-level registry (with `docs[]` and `skills[]`). Build merges it. Paths prefixed with author dir name.

### 2. Auto-discovery (no `registry.json`)

**DOC.md frontmatter:**
```yaml
---
name: openai-chat
description: OpenAI Chat API - completions, streaming, function calling
metadata:
  languages: "python,javascript,typescript"
  versions: "1.52.0"
  updated-on: "2026-01-15"
  source: maintainer
  tags: "openai,chat,llm"
---
```

**SKILL.md frontmatter** (no language/version):
```yaml
---
name: playwright-login
description: Login flow automation patterns for Playwright
metadata:
  updated-on: "2026-01-15"
  source: community
  tags: "browser,playwright,automation"
---
```

### Grouping logic
- DOC.md files with same `name` → grouped into one `docs[]` entry, each contributing languages/versions
- SKILL.md → one `skills[]` entry per unique name
- `files`: all files in the entry directory (relative paths)

## Example: different docs for different versions

```
openai/
└── docs/
    └── chat/
        ├── v1/
        │   ├── DOC.md              # versions: "1.52.0,1.51.0", languages: "python,javascript"
        │   └── references/
        │       └── streaming.md
        └── v2/
            ├── DOC.md              # versions: "2.0.0", languages: "python,javascript"
            └── references/
                ├── streaming.md
                └── structured-outputs.md
```

Both have `name: openai-chat` → grouped into one `docs[]` entry with multiple versions pointing to different paths.

## --full output behavior

- `--full` to stdout: concatenates all files with `# FILE:` headers
- `--full -o <dir>`: writes individual files preserving directory structure (so relative links resolve on disk)

## Build command

```bash
chub build <content-dir> [options]
```

Options:
- `-o, --output <dir>` — output directory (default: `<content-dir>/dist`)
- `--base-url <url>` — set `base_url` in registry
- `--validate-only` — check without writing
- `--json` — build summary as JSON

## Build steps

1. List top-level directories in `<content-dir>` (author directories)
2. For each author directory:
   a. If `registry.json` exists → read, validate, prefix paths
   b. Else → walk for DOC.md/SKILL.md, parse frontmatter, group into `docs[]` and `skills[]`
3. Merge all author entries (error on id collisions)
4. Write `registry.json` to output dir
5. Copy content tree to output dir
6. Print summary

## Validation rules

- DOC.md must have `name`, `description`, `metadata.languages`, `metadata.versions`
- SKILL.md must have `name`, `description` (no language/version required)
- Warn on missing `metadata.source` (default: "community")
- Error on id collision across different author directories
- If both DOC.md and SKILL.md in same directory, `name` must match
