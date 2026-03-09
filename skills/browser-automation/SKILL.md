---
name: Browser Automation
description: >
  Smart browser tool selector and automation orchestrator. Routes to the optimal browser
  tool based on task requirements. Default priority: Playwright MCP > Claude-in-Chrome > Puppeteer.
  Triggers on: "open browser", "navigate to", "test the UI", "screenshot", "fill form",
  "click button", "scrape", "E2E test", "visual QA", "browser automation", "check the site".
version: 1.0.0
---

# Browser Automation

You orchestrate browser automation by selecting the optimal tool for each task.

## Available Browser Tools (Priority Order)

### 1. Playwright MCP (Default — Best for Most Tasks)
- **Package:** `@playwright/mcp`
- **Add to Claude Code:** `claude mcp add playwright -- npx @playwright/mcp@latest`
- **Strengths:** Cross-browser (Chrome, Firefox, Safari, Edge), headless mode, device emulation, accessibility-tree based (token efficient), E2E testing, CI-compatible, Microsoft-maintained
- **Context cost:** ~13.7K tokens (21 tools)
- **Key flags:**
  - `--headless` — No visible browser (automation/CI)
  - `--browser chrome|firefox|webkit|msedge`
  - `--viewport-size 1280x720`
  - `--caps vision` — Enable screenshot-based vision
  - `--device "iPhone 15"` — Mobile emulation
  - `--user-data-dir <path>` — Persist login sessions
  - `--storage-state <file>` — Preload cookies/auth
- **Tools available:** browser_navigate, browser_click, browser_type, browser_drag, browser_select_option, browser_take_screenshot, browser_snapshot, browser_network_requests, browser_tab_list, browser_tab_new, browser_tab_close, browser_console_messages, browser_handle_dialog, browser_file_upload, browser_navigate_back, browser_navigate_forward, browser_wait
- **Auth approach:** In headed mode, Claude shows login page → you log in manually → cookies persist for session
- **Tip:** Explicitly mention "use playwright" in first request to ensure Claude uses the MCP instead of trying Bash

### 2. Claude-in-Chrome (For Logged-In Session Work)
- **Strengths:** Uses YOUR existing browser with all logged-in sessions, cookies, extensions. Perfect for interacting with authenticated pages you're already logged into.
- **Context cost:** ~18K tokens (26 tools)
- **When to use:** When you need to interact with sites where you're already authenticated (banking, admin panels, SaaS dashboards), or need Chrome extension functionality
- **Limitations:** Chrome only, requires extension, no headless, modal dialogs can freeze it, extension-dependent reliability
- **Tools:** mcp__claude-in-chrome__navigate, computer, find, form_input, get_page_text, javascript_tool, read_page, read_console_messages, read_network_requests, tabs_context_mcp, tabs_create_mcp, gif_creator, etc.

### 3. Puppeteer MCP (Legacy/Specific Use Cases)
- **Strengths:** Simple API, good for quick screenshots, Chrome/Chromium specific tasks
- **When to use:** Quick single-page screenshots, simple form fills where Playwright is unavailable
- **Tools:** puppeteer_navigate, puppeteer_screenshot, puppeteer_click, puppeteer_fill, puppeteer_hover, puppeteer_select, puppeteer_evaluate

## Decision Matrix

| Task | Best Tool | Why |
|------|-----------|-----|
| E2E testing | Playwright | Cross-browser, headless, deterministic |
| Visual QA / screenshots | Playwright | Device emulation, consistent viewport |
| Form testing | Playwright | Accessibility-tree based, reliable selectors |
| Scraping public sites | Playwright | Headless, fast, no extension needed |
| Interacting with logged-in SaaS | Claude-in-Chrome | Uses existing auth sessions |
| Admin panel operations | Claude-in-Chrome | Already logged in |
| Quick screenshot of live page | Playwright (headed) | Fast, reliable |
| CI/CD automation | Playwright | Headless, scriptable, no GUI needed |
| Mobile layout testing | Playwright | Device emulation (--device flag) |
| Performance/network analysis | Claude-in-Chrome | DevTools integration |
| Recording GIF of workflow | Claude-in-Chrome | gif_creator tool |
| PDF generation from page | Playwright | --caps pdf flag |

## Common Automation Recipes

### Visual QA — Check a deployed site
```
1. Use Playwright MCP to navigate to the URL
2. Take screenshots at key breakpoints (desktop, tablet, mobile via --device)
3. Check for visual regressions
4. Report findings
```

### E2E Test — Signup flow
```
1. Playwright: navigate to signup page
2. browser_snapshot to understand form structure
3. browser_type to fill fields
4. browser_click to submit
5. browser_snapshot to verify success state
```

### Scrape data from public site
```
1. Playwright (headless): navigate to URL
2. browser_snapshot for page structure
3. Extract data from accessibility tree
4. No screenshots needed — structured data is more efficient
```

### Interact with authenticated dashboard
```
1. Claude-in-Chrome: tabs_context_mcp to find existing tab
2. Or navigate to dashboard URL (already logged in)
3. read_page to understand current state
4. form_input / computer to interact
```

## Brain Integration

### Web Research → Intake
When scraping or researching for brain entities:
- Route extracted content to `_intake-dump/` for processing
- Tag with source URL and extraction date
- Link to relevant entity if applicable

### Visual QA → Project Tracking
When doing visual QA for brain projects:
- Store screenshots in project `_context/qa/`
- Log results to project status
- Create action items for issues found

## Setup Checklist

1. **Playwright MCP:** `claude mcp add playwright -- npx @playwright/mcp@latest`
2. **Claude-in-Chrome:** Install extension from Chrome Web Store (if not already)
3. **Puppeteer MCP:** Already configured if in ~/.claude.json

## Performance Notes

- Playwright uses accessibility tree snapshots (structured text) — far more token-efficient than screenshots
- Claude-in-Chrome uses page reading which can be verbose for complex pages
- For token optimization: prefer Playwright's browser_snapshot over screenshots when possible
- Headless Playwright is fastest — no rendering overhead
