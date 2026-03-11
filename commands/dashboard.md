---
name: dashboard
description: Launch the BizBrain OS visual dashboard in your browser — setup checklist, integrations, and quick actions
---

# /dashboard — BizBrain OS Visual Dashboard

Launch the visual dashboard — a local web app for managing your brain setup progress, integrations, and quick actions.

## Instructions

**You MUST automatically launch the dashboard. Do NOT just describe how — actually run the commands.**

1. **Check if the dashboard server is already running:**

```bash
curl -s http://localhost:3850/api/brain-status > /dev/null 2>&1 && echo "RUNNING" || echo "NOT_RUNNING"
```

2. **If RUNNING**, skip to step 4.

3. **If NOT running, find the dashboard, install deps, and start the server:**

```bash
# Find the dashboard directory inside the plugin
DASHBOARD_DIR=""

for dir in \
  "${CLAUDE_PLUGIN_ROOT}/tools/dashboard" \
  "$HOME/.claude/plugins/cache/TechIntegrationLabs/bizbrain-os-plugin/"*/tools/dashboard \
  "$HOME/.claude/plugins/cache/bizbrain-os-plugin/"*/tools/dashboard \
  "$HOME/Repos/bizbrain-os-plugin/tools/dashboard"; do
  if [ -d "$dir" ]; then
    DASHBOARD_DIR="$dir"
    break
  fi
done

if [ -z "$DASHBOARD_DIR" ]; then
  echo "ERROR: Dashboard not found in plugin directory"
  exit 1
fi

echo "Dashboard found at: $DASHBOARD_DIR"
```

Then install dependencies (if needed) and start the server:
```bash
cd "$DASHBOARD_DIR" && [ -d node_modules ] || npm install --silent 2>&1
node server.js &
```

Wait 2 seconds for the server to start.

4. **Open in the user's default browser:**

```bash
# Detect platform and open
if command -v start &>/dev/null || [ "$(uname -o 2>/dev/null)" = "Msys" ]; then
  start http://localhost:3850
elif command -v open &>/dev/null; then
  open http://localhost:3850
elif command -v xdg-open &>/dev/null; then
  xdg-open http://localhost:3850
fi
```

5. **Confirm to the user:**

```
BizBrain OS Dashboard is running at http://localhost:3850

Features:
  • Setup Checklist — 37 tasks across 8 categories to level up your brain
  • Integrations Hub — See connected vs available services
  • Quick Launch — Open brain, conversations, repos, or start Claude Code
  • Progress Tracking — Ring visualization with category breakdowns

The dashboard reads directly from your brain folder. All data stays local.
Close the terminal or press Ctrl+C to stop the server.
```

## Notes

- The dashboard runs on port 3850
- Dependencies auto-install on first launch (takes ~5 seconds)
- It serves a local web application — zero external dependencies after install
- All data is read from the brain folder on your machine
- The server auto-discovers the brain path (BIZBRAIN_PATH env, ~/bizbrain-os/, etc.)
- Each checklist item has an AI-generated preview icon, descriptions, benefits, and setup instructions
- Progress is saved to `<brain>/.bizbrain/dashboard/progress.json`
