---
name: design
description: Frontend design studio — orchestrate all design tools (UI UX Pro Max, page-design-guide, 21st.dev Magic, frontend-design)
user_invocable: true
---

# /design Command

Route to the `frontend-studio` skill for unified design intelligence.

## Usage

```
/design                    # Show available design tools and workflow
/design <description>      # Start design workflow for described project
/design review             # Run holistic design review on current project
/design component <desc>   # Generate a specific component via 21st.dev Magic
/design palette <mood>     # Get color palette recommendations
/design fonts <style>      # Get typography recommendations
```

## Routing

- Invoke the `frontend-studio` skill
- If user says "component" or "generate", route to 21st.dev Magic MCP
- If user says "review" or "audit", use Page Design Guide MCP holistic review
- If user says "palette" or "colors", use UI UX Pro Max color search + Page Design Guide
- Otherwise, run full design workflow from the skill
