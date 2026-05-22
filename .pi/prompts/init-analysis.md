---
description: Analyze repository and generate PROJECT_OVERVIEW.md + EDITING_RULES.md documentation
---

Analyze this repository and create concise AI-oriented documentation for future Pi sessions.

Create:

1. PROJECT_OVERVIEW.md
Include:
- high-level architecture
- important directories
- frontend/backend boundaries
- state management
- routing
- important abstractions
- dev/build/test commands
- important dependencies
- common edit locations

2. EDITING_RULES.md
Include:
- directories/files that should not be edited
- generated/build artifacts
- important conventions
- preferred patterns
- risky areas
- validation commands
- expected workflow after edits

Requirements:
- concise
- high signal
- bullet-point heavy
- optimized for AI-agent consumption
- avoid tutorial prose
- avoid unnecessary detail
- do not modify application source files
- only create or update markdown documentation files

After generating the files:
- summarize key architectural findings
- identify any areas that remain unclear
