---
name: Frontend Studio
description: >
  Unified frontend design intelligence — orchestrates UI UX Pro Max, frontend-design,
  page-design-guide MCP, and 21st.dev Magic MCP for comprehensive design workflows.
  Triggers on: /design, "design this", "UI for", "build frontend", "landing page",
  "dashboard design", "component library", "design system", "make it beautiful".
version: 1.0.0
---

# Frontend Studio

You orchestrate all available design tools for frontend work.

## Brain Location

Check: BIZBRAIN_PATH env → ~/bizbrain-os/
Design tools docs: `Knowledge/systems/ui-ux-design-tools.md`

## Available Design Tools

### 1. UI UX Pro Max (Plugin Skill)
- Searchable design database: 67 styles, 96 palettes, 57 fonts, 25 charts
- Invoke with: `/ui-ux-pro-max` or auto-triggers on UI tasks
- CLI: `python3 <brain>/Tools/ui-ux-pro-max/scripts/search.py "<query>" --domain <domain>`

### 2. Frontend Design (Plugin Skill)
- Distinctive aesthetic choices, bold typography/colors
- Invoke with: `/frontend-design`
- Best for: creative projects, landing pages, portfolios

### 3. Page Design Guide (MCP)
- Real-time design guidance via MCP tools
- Tools: `get_layout_patterns`, `get_color_guidance`, `get_typography_guidance`,
  `get_component_guidance`, `get_responsive_guidance`, `get_accessibility_guidance`,
  `get_animation_guidance`, `get_section_guidance`, `get_modern_trends`,
  `get_modern_palettes`, `get_inspiration_by_mood`, `get_holistic_design_review`

### 4. 21st.dev Magic (MCP)
- AI-generated UI components from natural language
- Prefix requests with `/ui` for component generation
- Generates React/Tailwind/shadcn components

## Workflow

When a user asks for frontend/design work:

1. **Understand the project type** — SaaS, landing page, dashboard, e-commerce, portfolio, mobile
2. **Gather design direction** — Ask about mood, style preferences, brand colors if not specified
3. **Research phase:**
   - Use Page Design Guide MCP for layout patterns and section guidance
   - Use UI UX Pro Max to search relevant styles, palettes, and typography
4. **Design system setup:**
   - Define colors, typography, spacing based on research
   - Use `get_modern_palettes` for trending options
5. **Component generation:**
   - Use 21st.dev Magic for complex components (`/ui <description>`)
   - Apply Frontend Design skill for distinctive aesthetic choices
6. **Review:**
   - Run `get_holistic_design_review` for full audit
   - Check `get_accessibility_guidance` for a11y compliance

## Quick Commands

| Request | Action |
|---------|--------|
| "Make it beautiful" | Apply Frontend Design + modern palette + distinctive fonts |
| "Design a dashboard" | UI UX Pro Max (SaaS style) + Page Design Guide (layout) |
| "Landing page for X" | Full workflow: research → design system → components |
| "Review the design" | Page Design Guide holistic review + accessibility audit |
| "Generate a component" | Route to 21st.dev Magic MCP |

## Integration with Content Pipeline

When building frontends for brain projects:
- Check `Projects/<name>/_context/` for brand guidelines
- Check `Clients/<name>/_context/` for client preferences
- Store generated design systems in project's `_context/design-system.md`
