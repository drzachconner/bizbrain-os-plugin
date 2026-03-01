---
name: swarm
description: Brain Swarm orchestration — view status, process events, resolve conflicts, and manage the orchestration layer
---

# /swarm — Brain Swarm Management

Manage the Brain Swarm orchestration layer. Requires `features.orchestration: true` in brain config.

## Subcommands

### `/swarm` or `/swarm status`

Show the current state of the orchestration layer:

```
Brain Swarm Status
──────────────────
  Event Queue:     3 pending
  Staging:         1 pending, 0 conflicts
  Last Changelog:  2 entries today
  Patterns:        6 active (3 high confidence)

Recent Changes:
  14:31 — brain-learner added action item to BuildTrack
  14:28 — entity-watchdog updated Acme Corp contact
  14:25 — brain-learner logged architecture decision
```

**Implementation:**
1. Read brain config, verify `features.orchestration` is true
2. Count files in `.bizbrain/events/` (excluding `_processed/`)
3. Count files in `.bizbrain/staging/pending/` and `.bizbrain/staging/conflicts/`
4. Read today's changelog from `.bizbrain/changelog/YYYY-MM-DD.md`
5. Read patterns from `Operations/learning/patterns/workflows.json`
6. Present summary

### `/swarm process`

Manually trigger event queue processing:

1. Invoke the `brain-orchestrator` agent
2. Process all queued events
3. Validate all pending staged proposals
4. Report results: "Processed N events, applied M changes, K conflicts"

### `/swarm changelog [date]`

View the changelog for a specific date (default: today):

```
/swarm changelog           → Today's changes
/swarm changelog 2026-02-28 → Changes from Feb 28
/swarm changelog week       → Last 7 days summary
```

**Implementation:**
1. Read `.bizbrain/changelog/{date}.md`
2. If "week", read last 7 daily files and present combined summary
3. Present formatted changelog

### `/swarm patterns`

View and manage workflow patterns:

```
Workflow Patterns (6 active)
────────────────────────────
  [1] new_project_onboarding    ★★★★☆  (0.90)  Used: 4 times
  [2] entity_mention_update     ★★★★☆  (0.85)  Used: 12 times
  [3] decision_capture          ★★★★☆  (0.80)  Used: 7 times
  [4] session_wrapup            ★★★★★  (0.90)  Used: 15 times
  [5] action_item_extraction    ★★★★☆  (0.85)  Used: 9 times
  [6] client_meeting_debrief    ★★★☆☆  (0.75)  Used: 2 times
```

**Implementation:**
1. Read `Operations/learning/patterns/workflows.json`
2. Sort by `times_used` descending
3. Present with confidence stars and usage counts

### `/swarm conflicts`

View and resolve staging conflicts:

```
Staging Conflicts (1)
─────────────────────
Conflict #1: Projects/BuildTrack/action-items.md
  [A] brain-learner (14:31): Add "Deploy auth fix by Friday"
  [B] brain-learner (14:32): Add "Review PR #42 before deploy"

  Actions:
    [1] Apply A only
    [2] Apply B only
    [3] Apply both (merge)
    [4] Discard both
```

**Implementation:**
1. Read all files in `.bizbrain/staging/conflicts/`
2. Group by `target_path`
3. Present each conflict with options
4. Apply user's choice → move to `_processed/` → write changelog

## Error Handling

If orchestration is not enabled:
```
Brain Swarm is not enabled. Enable it with:
  1. Edit your brain config.json
  2. Set features.orchestration to true
  3. Restart Claude Code

Or run: /brain configure → toggle orchestration
```
