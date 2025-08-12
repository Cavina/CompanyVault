# Company Vault — Obsidian PM (GitHub-backed)

Lightweight PM system for small teams using Obsidian.
Core ideas: **atomic tasks** (one file per task), **dashboards/boards**, GitHub as the **source of truth**.

## Folder layout
- \`tasks/<assignee>/\` — one Markdown file per task (atomic)
- \`projects/<project>/\` — specs, briefs
- \`dashboards/\` — saved views (Dataview/Projects)
- \`templates/\` — Templater templates
- \`.obsidian/\` — shared plugin configs (not workspace state)

## Task frontmatter (schema)
\`\`\`yaml
---
id: 2025-08-11T12-34-56
title: Implement audio stream FSM
assignee: alice
status: todo           # todo | in-progress | review | done
due: 2025-08-15
project: multimedia-app
tags: [priority-high]
created: 2025-08-11
updated: 2025-08-11
---
\`\`\`

## Recommended plugins
Dataview • Templater • Projects (optional) • Obsidian Git • Calendar (optional)

## Obsidian Git (suggested)
- Auto pull on startup
- Auto commit on file change (or interval)
- Auto push every 5–10 min

## Getting started
1) Open this repo as an Obsidian vault.
2) Install the plugins above.
3) Use \`templates/task.md\` to create tasks.
4) Open \`dashboards/\` notes for team views.
