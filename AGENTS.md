# BizBrain-OS — Agent Context

## What This Project Is

BizBrain OS is a Claude Code plugin (v3.3.1) that solves "context amnesia" — re-explaining your projects, clients, and tech stack at the start of every session. It automatically injects your full business context at session start from a local-only brain folder (`~/bizbrain-os/`). The brain compounds over time: decisions, entities, patterns, and action items are captured continuously.

Repo: `TechIntegrationLabs/bizbrain-os-plugin` (Claude Code plugin marketplace)

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Plugin system | Claude Code Plugin API (v3.3.1) |
| Hooks | SessionStart, PostToolUse, SessionEnd (`hooks/hooks.json`) |
| Context generation | Node.js 18+ (`scripts/generate-context.js`) |
| Machine scanning | Bash (`scripts/scanner.sh`) |
| Visual dashboard | Express.js (port 3850) + vanilla JS/HTML SPA (`tools/dashboard/`) |
| Meeting transcription | Python 3.10+ with faster-whisper (`tools/meeting-transcriber/`) |

## Architecture

**Three-pillar design:**

```
Pillar 1: Brain Folder (local-only)
  ~/bizbrain-os/
  ├── launchpad/    — Daily driver zone (~120 lines context)
  ├── workspaces/   — Code repos with lean context (~80 lines)
  └── brain/        — Full business intelligence (~300 lines)

Pillar 2: Automatic Injection
  SessionStart hook → generate-context.js → context string injected into Claude

Pillar 3: Continuous Learning
  PostToolUse hook → event queue → entity watchdog + brain learner
```

**Hooks pipeline:** `SessionStart → session-start → context injection` / `PostToolUse → post-tool-use → learning + time tracking` / `SessionEnd → session-end → archive trigger`

**Brain Swarm** (opt-in, `"orchestration": true` in brain's `config.json`): orchestrator → agents → staging → validation → brain → changelog. Off by default.

## Directory Structure

```
BizBrain-OS/
├── .claude-plugin/
│   ├── plugin.json          # Plugin metadata
│   └── marketplace.json     # Marketplace listing
├── agents/                  # 4 background agent specs (.md)
├── commands/                # 17 slash commands (.md)
├── docs/                    # Getting started guide, dashboard design
├── hooks/
│   ├── hooks.json           # Hook definitions
│   ├── run-hook.cmd         # Cross-platform polyglot (Windows + Unix)
│   └── scripts/             # Hook implementation scripts
├── lib/
│   ├── default-config.json  # Brain config template
│   ├── folder-structure.json
│   ├── integrations-registry.json  # 37+ service integrations
│   ├── seed-patterns.json
│   └── zone-templates/      # CLAUDE.md templates per zone
├── profiles/                # 5 role-based profile JSONs
├── scripts/
│   ├── generate-context.js  # SessionStart context builder (core logic)
│   └── scanner.sh           # Machine discovery script
├── skills/                  # 23 skill packages (SKILL.md per skill)
└── tools/
    ├── dashboard/            # Express server (port 3850) + SPA frontend
    └── meeting-transcriber/  # Python: local audio transcription
```

## Conventions

- **No `package.json` at root** — this is a Claude Code plugin, not a Node package. Only `tools/dashboard/` has `package.json`
- **Skill structure**: `skills/<name>/SKILL.md` with YAML frontmatter (`name`, `description`, `version`) — instruction documents only, no code files
- **Agent structure**: `agents/*.md` with YAML frontmatter (`name`, `description`, `model`, `color`, `tools`) + system prompt prose
- **Command structure**: `commands/*.md` with YAML frontmatter (`name`, `description`, `argument-hint`)
- **Profile structure**: JSON files in `profiles/` with feature toggles, auto-behaviors, scan targets
- **Brain config**: Generated at `~/bizbrain-os/config.json` during setup — never stored in this repo

## Key Files

- `scripts/generate-context.js` — Core logic for SessionStart context injection
- `hooks/hooks.json` — Hook definitions that wire the plugin to Claude Code events
- `hooks/run-hook.cmd` — Cross-platform hook runner (Windows + Unix polyglot)
- `lib/default-config.json` — Default brain configuration template
- `lib/integrations-registry.json` — 37+ external service integration catalog
- `tools/dashboard/server.js` — Express API server on port 3850

## Available Profiles

| Profile | Use Case |
|---------|----------|
| `developer` | Repos, GSD workflow, credentials, entity management |
| `agency` | All features — clients, billing, content, outreach, team |
| `consultant` | Client entities, proposals, time tracking, billing |
| `content-creator` | Content pipeline, social scheduling, outreach |
| `personal` | Minimal — knowledge base, todos, intake processing |

## Adding New Content

**New skill:** Create `skills/<name>/SKILL.md` with YAML frontmatter, then update README feature table.

**New command:** Create `commands/<name>.md` with YAML frontmatter. Command invoked as `/<name>`.

**New agent:** Create `agents/<name>.md` with YAML frontmatter including `model` field.

**New feature (large):** Follow this sequence — skill spec → command → agent (if needed) → update all 5 profile JSONs + `lib/default-config.json` → update README.

## What NOT to Touch

- **`~/bizbrain-os/`** — the user's brain folder lives outside this repo; never modify it programmatically from within the repo
- **`tools/dashboard/public/`** — generated SPA assets; edit source, not output
- **`lib/integrations-registry.json`** — do not remove existing integrations; only append new ones
- **Brain folder gitignored by design** — `~/bizbrain-os/` is outside any repo; API keys for integrations should use 1Password

## Known Issues

- Meeting transcription `tools/meeting-transcriber` uses WASAPI loopback (Windows-specific); macOS/Linux need different audio capture
- Full-mode brain injects ~300 lines/session — use launchpad mode if context is tight
- Brain Swarm adds staging overhead to every tool call; leave disabled unless needed
- Dashboard brain discovery fixed in v3.3.1 — upgrade if dashboard shows empty state

## Related

- See `CLAUDE.md` for Claude Code-specific configuration
