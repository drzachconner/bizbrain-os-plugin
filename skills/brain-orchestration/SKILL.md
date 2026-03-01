---
name: brain-orchestration
description: |
  Brain Swarm orchestration layer for BizBrain OS. Coordinates all brain agents
  through an event queue, staging area with validation, changelog audit trail,
  workflow pattern learning, and smart model routing.
version: 1.0.0
---

# Brain Orchestration — Brain Swarm

The orchestration layer that turns independent brain agents into a coordinated swarm.

## Architecture Overview

```
User Session
    │
    ├── SessionStart ──→ Orchestrator Brief (queue depth, conflicts, recent changes)
    │
    ├── PostToolUse ───→ Event Queue (.bizbrain/events/)
    │                         │
    │                    Orchestrator
    │                    ┌────┴────┐
    │               Pattern    Routing
    │               Match      Table
    │                    │         │
    │              ┌─────┴─────────┴─────┐
    │              ▼         ▼           ▼
    │         Watchdog   Learner    Gateway
    │              │         │           │
    │              └─────┬───┘           │
    │                    ▼               │
    │              Staging Area          │
    │              ┌─────────┐           │
    │              │ Pending  │           │
    │              │ Conflicts│           │
    │              └────┬────┘           │
    │                   ▼                │
    │              Validation            │
    │                   │                │
    │                   ▼                │
    │              Brain Files ◄─────────┘
    │                   │
    │                   ▼
    │              Changelog
    │
    └── SessionEnd ───→ Session Summary Event
```

## Event Queue

### Location
`.bizbrain/events/`

### Event File Format
Filename: `{timestamp}-{type}-{short-id}.json`

Example: `2026-03-01T14-30-00Z-tool_use-a1b2.json`

```json
{
  "id": "evt_a1b2c3d4",
  "type": "tool_use",
  "timestamp": "2026-03-01T14:30:00Z",
  "source": "post-tool-use",
  "data": {
    "tool_name": "Write",
    "file_path": "/path/to/file",
    "project": "my-project"
  },
  "processed": false
}
```

### Event Types
| Type | Source | Triggers |
|------|--------|----------|
| `tool_use` | PostToolUse hook | Every tool invocation (Write, Edit, Bash, etc.) |
| `entity_mention` | Context injection | Entity name detected in conversation |
| `decision` | Context injection | User makes a decision |
| `action_item` | Context injection | Task/deadline mentioned |
| `session_event` | SessionStart/End | Session lifecycle events |

### Processing Rules
1. Read all `.json` files in `.bizbrain/events/` (not in `_processed/`)
2. Sort by timestamp (oldest first)
3. For each event, attempt workflow pattern match
4. If pattern matches → execute sequence
5. If no match → route via routing table
6. Move processed event to `.bizbrain/events/_processed/`
7. Write changelog entry

## Staging Area

### Location
`.bizbrain/staging/`

### Proposal Format
Filename: `{timestamp}-{agent}-{short-id}.json`

```json
{
  "id": "stg_x1y2z3",
  "agent": "brain-learner",
  "timestamp": "2026-03-01T14:31:00Z",
  "action": "append",
  "target_path": "Projects/my-project/action-items.md",
  "content": "- [ ] AI-042: Deploy auth fix by Friday (source: session 2026-03-01)",
  "reason": "Action item extracted from conversation",
  "urgency": "normal"
}
```

### Validation Pipeline
```
Pending Proposal
    │
    ├── Conflict Check ──→ Same target_path? → staging/conflicts/
    │
    ├── Duplicate Check ──→ Content exists? → Discard + changelog
    │
    ├── Staleness Check ──→ Target modified after proposal? → Flag for review
    │
    ├── Auth Check ──→ New entity create? → Require user confirmation
    │
    └── Valid ──→ Apply to brain → staging/_processed/ → Changelog
```

### Conflict Resolution
When two proposals target the same file:
1. Both moved to `staging/conflicts/`
2. `/swarm conflicts` presents them to the user
3. User picks one, merges, or discards
4. Winner applied → changelog records resolution

## Changelog

### Location
`.bizbrain/changelog/{YYYY-MM-DD}.md`

### Format
```markdown
# Brain Changelog — 2026-03-01

## 14:31:05 — brain-learner append Projects/my-project/action-items.md
- **What:** Added action item AI-042: Deploy auth fix by Friday
- **Why:** Extracted from conversation
- **Source:** evt_a1b2c3d4

## 14:32:10 — entity-watchdog update Entities/Clients/Acme/_meta.json
- **What:** Updated primary contact email
- **Why:** User mentioned new email address
- **Source:** evt_e5f6g7h8

## 14:33:00 — brain-orchestrator discard stg_duplicate_01
- **What:** Discarded duplicate action item (already exists)
- **Why:** Content matches existing entry in action-items.md
- **Source:** stg_x1y2z3
```

## Workflow Patterns

### Location
`Operations/learning/patterns/workflows.json`

### Pattern Format
```json
{
  "id": "pattern_id",
  "trigger": "Human-readable description of when this fires",
  "sequence": ["agent:action", "agent:action", ...],
  "confidence": 0.85,
  "times_used": 12
}
```

### Pattern Learning
- After a successful multi-agent sequence, record it as a new pattern
- Increment `times_used` on pattern match + success
- Decrease `confidence` by 0.1 for unused patterns (monthly)
- Remove patterns with `confidence < 0.3`

## Model Routing Table

| Task | Agent | Tier | Why |
|------|-------|------|-----|
| Entity detection | entity-watchdog | haiku | Pattern matching is fast |
| Decision logging | brain-learner | haiku | Structured extraction |
| Project onboarding | brain-learner | haiku | Repo read + metadata write |
| Session summary | brain-learner | haiku | Concise summarization |
| Conflict resolution | brain-orchestrator | haiku | Rule-based validation |
| Complex brain ops | brain-gateway | sonnet | Full business intelligence |

**Cost impact:** Routing simple tasks to haiku saves 40-60% on agent operations vs. running everything on sonnet.

## Backward Compatibility

- **Orchestration OFF (default):** Zero behavior change. No events, no staging, no orchestrator context. Agents write directly to brain as in v3.0.1.
- **Orchestration ON:** Events are queued, agents write to staging, orchestrator validates and applies. Full changelog audit trail.
- **Switching:** Toggle `features.orchestration` in config.json. Existing brain data is unaffected. Event/staging directories are created on first use.

## Pruning Schedule

| Data | Retention | Action |
|------|-----------|--------|
| Processed events | 7 days | Auto-delete from `_processed/` |
| Applied proposals | 14 days | Auto-delete from `_processed/` |
| Conflict proposals | Until resolved | User must resolve or discard |
| Changelog entries | Indefinite | Audit trail, never auto-delete |
| Workflow patterns | Confidence-based | Decay unused, remove < 0.3 |
