# BizBrain OS Plugin

## Project Overview

BizBrain OS is a Claude Code plugin (v3.3.1) that creates a persistent, local-only knowledge brain for Claude Code sessions. It solves "context amnesia" — the problem of re-explaining your projects, clients, and tech stack at the start of every session — by automatically injecting your full business context at session start. The brain compounds over time: decisions, entities, patterns, and action items are captured continuously and made available in every future session.

## Workflow Rules

- **ALWAYS pull before working**: Run `git pull --rebase` before making any changes. This is mandatory for multi-machine sync.
- **ALWAYS commit and push after making changes.** After completing ANY code changes, immediately stage modified files by name, commit with a descriptive message, and push. Every change must end with a successful `git push`.
- **Never leave files behind.** Before ending any session, run `git status` and confirm zero untracked or modified files.
- Never use `git add .` or `git add -A` — always add specific files by name.
- Commit message format: conventional commits (feat:, fix:, chore:, docs:). Always include `Co-Authored-By: Claude <noreply@anthropic.com>`.

## Tech Stack

- **Plugin System:** Claude Code Plugin API (v3.3.1)
- **Hooks:** SessionStart, PostToolUse, SessionEnd (via `hooks/hooks.json`)
- **Context Generation:** Node.js 18+ (`scripts/generate-context.js`)
- **Machine Scanning:** Bash (`scripts/scanner.sh`)
- **Visual Dashboard:** Express.js server (port 3850) + vanilla JS/HTML SPA (`tools/dashboard/`)
- **Meeting Transcription:** Python 3.10+ with faster-whisper (`tools/meeting-transcriber/`)
- **License:** AGPL-3.0

## Architecture

### Three-Pillar Design

```
Pillar 1: Brain Folder (local-only knowledge store)
  ~/bizbrain-os/
  ├── launchpad/       — Daily driver zone (~120 lines of context)
  ├── workspaces/      — Code repos with lean context (~80 lines)
  └── brain/           — Full business intelligence (~300 lines)
      ├── Knowledge/   — Systems, decisions, templates, reports
      ├── Entities/    — Clients, Partners, Vendors
      ├── Projects/    — Tracked repos with status/stack/links
      ├── Operations/  — Credentials, todos, timesheets, learning
      └── _intake-dump/ — Drop zone for voice notes, PDFs, transcripts

Pillar 2: Automatic Injection (SessionStart hook → generate-context.js)
  — Reads brain config + state → builds context string → injects into Claude

Pillar 3: Continuous Learning (PostToolUse hook → event queue)
  — Every tool call timestamps for time tracking
  — Entity Watchdog detects entity mentions, auto-updates records
  — Brain Learner captures decisions, action items, session summaries
```

### Hooks Pipeline

```
SessionStart  → scripts/session-start  → context injection
PostToolUse   → scripts/post-tool-use  → learning + time tracking
SessionEnd    → scripts/session-end    → session archive trigger
```

### Brain Swarm (opt-in orchestration)

```
User Session → PostToolUse → Event Queue → Orchestrator → Agents → Staging → Validation → Brain → Changelog
```

Enable with `"orchestration": true` in brain's `config.json`. Off by default.

### Four Background Agents

| Agent | Role | Model Tier |
|-------|------|------------|
| `brain-orchestrator` | Event queue, staging validation, conflict detection, changelog | haiku |
| `entity-watchdog` | Auto-detect entity mentions in conversation, update records | haiku |
| `brain-learner` | Capture decisions, action items, patterns, session summaries | haiku |
| `brain-gateway` | Full brain access from any repo or directory | sonnet |

### Five Profiles

| Profile | When to Use |
|---------|-------------|
| `developer` | Repos, GSD workflow, credentials, entity management |
| `agency` | All features — clients, billing, content, outreach, team tracking |
| `consultant` | Client entities, proposals, time tracking, communications, billing |
| `content-creator` | Content pipeline, social scheduling, outreach, audience management |
| `personal` | Minimal — knowledge base, todos, intake processing |

## Directory Structure

```
BizBrain-OS/
├── .claude-plugin/
│   ├── plugin.json          # Plugin metadata (name, version, author)
│   └── marketplace.json     # Marketplace listing
├── agents/                  # 4 background agent specs
│   ├── brain-orchestrator.md
│   ├── brain-learner.md
│   ├── brain-gateway.md
│   └── entity-watchdog.md
├── assets/                  # Images for README + docs
├── commands/                # 17 slash commands
│   ├── brain.md             # /brain — status, setup, scan, configure, profile
│   ├── dashboard.md         # /dashboard — visual browser dashboard
│   ├── entity.md            # /entity — client/partner/vendor lookup
│   ├── todo.md              # /todo — unified task management
│   ├── knowledge.md         # /knowledge — brain search
│   ├── hours.md             # /hours — time tracking
│   ├── intake.md            # /intake — file processing from _intake-dump
│   ├── mcp.md               # /mcp — MCP server management
│   ├── gsd.md               # /gsd — project execution (milestones/phases/waves)
│   ├── meetings.md          # /meetings — local meeting transcription
│   ├── swarm.md             # /swarm — brain swarm orchestration
│   ├── comms.md             # /comms — communication hub
│   ├── content.md           # /content — content pipeline
│   ├── outreach.md          # /outreach — lead pipeline
│   ├── design.md            # /design — frontend studio
│   ├── download.md          # /download — media downloader
│   └── archive-sessions.md  # /archive-sessions — Obsidian export
├── docs/
│   ├── BIZBRAIN-OS-GETTING-STARTED-GUIDE.md
│   └── DASHBOARD-DESIGN.md
├── hooks/
│   ├── hooks.json           # Hook definitions (SessionStart/PostToolUse/SessionEnd)
│   ├── run-hook.cmd         # Cross-platform polyglot (Windows + Unix)
│   └── scripts/             # Hook implementation scripts
│       ├── session-start
│       ├── post-tool-use
│       └── session-end
├── lib/
│   ├── default-config.json  # Brain config template
│   ├── folder-structure.json # Brain folder definitions
│   ├── integrations-registry.json # 37+ service integrations registry
│   ├── seed-patterns.json   # Initial workflow patterns for orchestrator
│   └── zone-templates/      # CLAUDE.md templates per zone (launchpad/workspaces/brain)
├── profiles/                # 5 role-based profile JSONs
│   ├── agency.json
│   ├── consultant.json
│   ├── content-creator.json
│   ├── developer.json
│   └── personal.json
├── scripts/
│   ├── generate-context.js  # SessionStart context builder (Node.js, 35k lines)
│   └── scanner.sh           # Machine discovery script
├── skills/                  # 23 skill packages (SKILL.md per skill)
│   ├── brain-bootstrap/     # /brain setup wizard
│   ├── brain-learning/      # Continuous context capture
│   ├── brain-orchestration/ # Swarm coordination
│   ├── browser-automation/  # Smart browser tool selector
│   ├── communications/      # Email/Slack/Discord tracking
│   ├── content-pipeline/    # Ideation → drafting → scheduling
│   ├── credential-management/ # Secure local vault
│   ├── entity-management/   # Entity CRUD + watchdog rules
│   ├── event-asset-studio/  # Event collateral generation
│   ├── frontend-studio/     # UI/UX design work
│   ├── google-workspace/    # Gmail, Calendar, Drive via CLI + MCP
│   ├── gsd-workflow/        # Phase/wave/task execution
│   ├── intake-processing/   # File classification + routing
│   ├── knowledge-management/ # Systems, decisions, templates
│   ├── mcp-management/      # Profile-based MCP switching
│   ├── media-downloader/    # yt-dlp, ffmpeg media acquisition
│   ├── meeting-transcription/ # Local audio → text
│   ├── outreach-engine/     # Lead pipeline + sequences
│   ├── project-tracking/    # Repo discovery + status
│   ├── session-archiving/   # Obsidian export with frontmatter
│   ├── social-engage/       # Community engagement (newest)
│   ├── time-tracking/       # Heartbeat-based logging
│   └── todo-management/     # Cross-source task aggregation
├── tools/
│   ├── dashboard/           # Visual dashboard (Express + vanilla JS)
│   │   ├── server.js        # API server on port 3850
│   │   ├── generate-icons.js # AI icon generation
│   │   └── public/          # SPA frontend (HTML/CSS/JS)
│   └── meeting-transcriber/ # Python: local audio transcription
├── .gitignore
└── README.md
```

## Development Conventions

- **No package.json at root** — this is a Claude Code plugin, not a Node package. The only `package.json` is inside `tools/dashboard/`.
- **Skill structure**: Each skill directory contains a `SKILL.md` with YAML frontmatter (`name`, `description`, `version`) followed by instruction prose. No code files — skills are instruction documents.
- **Agent structure**: Agent files in `agents/` are markdown with YAML frontmatter (`name`, `description`, `model`, `color`, `tools`) followed by system prompt prose.
- **Command structure**: Command files in `commands/` are markdown with YAML frontmatter (`name`, `description`, `argument-hint`) — invoked as `/command-name`.
- **Profile structure**: JSON files in `profiles/` define feature toggles, auto-behaviors, scan targets, and recommended integrations.
- **Brain config**: Generated at `~/bizbrain-os/config.json` during setup; never stored in this repo.
- **Cross-platform hooks**: `hooks/run-hook.cmd` is a polyglot that runs on both Windows (batch) and Unix (bash).

## Environment Variables

The plugin itself requires no `.env` file. The following are used at runtime (set by the user's shell environment, not this repo):

- `BIZBRAIN_PATH` — Override brain folder location (default: `~/bizbrain-os/`)
- `CLAUDE_PLUGIN_ROOT` — Set automatically by Claude Code plugin system; points to this repo

The integrations registry (`lib/integrations-registry.json`) documents 37+ external services users may configure. Their API keys are never stored in this repo — BizBrain OS catalogs them locally in the brain's `Operations/credentials/` folder.

## Workflow

### Installation (end users)

```bash
claude plugin marketplace add TechIntegrationLabs/bizbrain-os-plugin
claude plugin install bizbrain-os
# In a new terminal:
claude
> /brain setup
> /dashboard
```

### Development (this repo)

```bash
# Clone and inspect
git clone https://github.com/TechIntegrationLabs/bizbrain-os-plugin
cd bizbrain-os-plugin

# Run the visual dashboard locally (for dashboard development)
cd tools/dashboard
npm install
node server.js
# Opens on http://localhost:3850

# Test meeting transcriber (optional)
cd tools/meeting-transcriber
pip install -e .

# Add a new skill
mkdir skills/my-skill
# Create skills/my-skill/SKILL.md with YAML frontmatter

# Add a new command
# Create commands/my-command.md with YAML frontmatter
```

### Key Commands (after installation)

```
/brain setup      First-time wizard
/brain scan       Re-discover machine projects
/brain profile    Switch profile (developer/agency/consultant/creator/personal)
/dashboard        Visual browser dashboard (port 3850)
/entity <name>    Look up client, partner, or vendor
/todo             Unified task dashboard
/knowledge <q>    Search brain knowledge base
/hours            Time tracking summary
/gsd              Project execution (milestones → phases → waves → tasks)
/intake           Process files in _intake-dump/
/swarm            Brain Swarm orchestration (if enabled)
/meetings         Local meeting transcription
/comms            Communication hub
/content          Content pipeline
/outreach         Lead pipeline
```

## Known Issues

1. **Meeting transcription macOS/Linux gap**: `tools/meeting-transcriber` docs mention WASAPI loopback (Windows-specific). macOS/Linux users need a different audio capture approach.
2. **Dashboard brain discovery**: Fixed in v3.3.1 (`5abfc6d`) — `server.js` now correctly discovers brain in full mode. Upgrade to latest if dashboard shows empty state.
3. **Context window cost**: Full-mode brain (`~/bizbrain-os/brain/`) injects ~300 lines per session. Heavy users may want compact mode (~120 lines from launchpad) or selective feature disabling in `config.json`.
4. **Swarm disabled by default**: `"orchestration": false` in `default-config.json`. Enable explicitly — it adds staging validation overhead to every tool use.
5. **NotebookLM "Skills & Tools" source too large**: If uploading brain content to NotebookLM, the consolidated skills export may exceed the 500k word limit. Split by category.

## Security

- **Brain folder is 100% local** — never uploaded by the plugin. Users control where it lives via `BIZBRAIN_PATH`.
- **Credentials cataloged, never auto-copied** — `credential-management` skill masks values in display (`ghp_...abc` format) and never reads/writes to `.env` files automatically.
- **No telemetry** — the plugin makes no external API calls of its own. External service calls are always user-initiated via commands.
- **Brain folder gitignored** — `~/bizbrain-os/` is outside any repo by design. Users who git-track their brain should ensure `Operations/credentials/` is excluded.
- **This repo**: No secrets. No `.env` needed. `.gitignore` covers `node_modules/`, Python artifacts, and `.DS_Store`.
- **API keys for integrations**: Use 1Password to store. Reference via `op item get` when needed — never hardcode in any file in this repo.

## Subagent Orchestration

| Subagent | When to Deploy |
|----------|---------------|
| **codebase-explorer** | Before modifying hook scripts, `generate-context.js`, or `scanner.sh` — understand how context injection and event queuing interact |
| **docs-weaver** | After adding a new skill or command — update README feature tables and getting-started guide |
| **code-review** | Before PRs — verify skill SKILL.md frontmatter is valid, command argument-hints are correct, profile JSONs have required fields |
| **browser-navigator** | For end-to-end testing of the visual dashboard (`/dashboard` command + Express server on port 3850) |
| **security-scanner** | After any change to credential-management skill or hooks that touch `Operations/credentials/` |

## GSD + Teams Strategy

This is a **plugin source repo** with a simple contribution model — add skill, add command, tweak hook logic. Tasks are generally small and sequential. GSD phases are not needed for routine contributions.

For larger feature work (e.g., a new integration category, major hook refactor), structure as:

```
Phase 1: Skill spec — write SKILL.md + any reference docs
Phase 2: Command — write commands/my-feature.md
Phase 3: Agent (if needed) — write agents/my-agent.md
Phase 4: Profile updates — add feature toggle to all 5 profile JSONs + default-config.json
Phase 5: README — update feature table, commands list, architecture diagram
```

Teams parallelism is rarely needed — skills, commands, and agents are independent files with no shared state. Exception: if adding a feature that touches hooks + lib/ + skills/ simultaneously, spawn two teammates (one for hooks/lib, one for skills/commands).

## MCP Connections

The plugin itself has no MCP dependencies. However, `lib/integrations-registry.json` documents 37+ services with guided MCP setup, and the `mcp-management` skill handles profile-based MCP switching for users.

**User-facing MCP integrations** (configured by end users in their brain, not in this repo):
- GitHub, Supabase, Stripe, Netlify, Vercel (Development)
- Slack, Discord, Gmail (Communication)
- Notion, Google Workspace (Productivity)
- Playwright MCP, Puppeteer (Browser automation)
- Postiz, Late.dev, Buffer (Publishing)
- Firecrawl, Screenpipe, GoHighLevel (Research/CRM)

## Completed Work

- **v3.3.1** (current): Bug fix for dashboard brain discovery in full mode; `social-engage` skill added
- **v3.3.0**: Visual Dashboard — Express server on port 3850, 37 AI-generated icons, setup checklist, integrations hub, progress ring, quick launch
- **v3.2.0**: Dashboard SPA frontend, category progress bars, detail pages per integration
- **v3.1.0**: Google Workspace skill (Gmail/Calendar/Drive via CLI + MCP), Browser Automation skill (smart tool selector: Playwright → Claude-in-Chrome → Puppeteer)
- **Prior**: Brain Swarm orchestration, meeting transcription (faster-whisper), `frontend-studio` skill, `media-downloader` skill, cross-platform Windows hook support
- **23 skills** installed: brain-bootstrap, brain-learning, brain-orchestration, browser-automation, communications, content-pipeline, credential-management, entity-management, event-asset-studio, frontend-studio, google-workspace, gsd-workflow, intake-processing, knowledge-management, mcp-management, media-downloader, meeting-transcription, outreach-engine, project-tracking, session-archiving, social-engage, time-tracking, todo-management
- **17 commands**: brain, dashboard, entity, todo, knowledge, hours, intake, mcp, gsd, meetings, swarm, comms, content, outreach, design, download, archive-sessions
- **4 background agents**: brain-orchestrator, brain-learner, brain-gateway, entity-watchdog
- **5 profiles**: developer, agency, consultant, content-creator, personal
- **37+ integrations** in registry across development, communication, social, productivity, browser, AI, publishing, research, CRM categories
