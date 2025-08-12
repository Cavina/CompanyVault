# Obsidian PM — Architecture & Relationships (Companion Notes)

**Purpose:** atomic tasks, two interaction modes (direct file editing vs dashboards), GitHub as source of truth.

## Components
- **Clients:** Manager, Employees; interfaces: Tasks hub, Dashboards hub
- **Shared Vault:** `tasks/`, `projects/`, `dashboards/`, `templates/`, `.obsidian/`; hubs: Task Data, Dashboard Data
- **Plugins:** Input (from dashboards), Dataview, Kanban, Projects (opt), Templater/Calendar/Git, Output (all views)
- **GitHub Remote:** `origin/main`
- **Automations:** nightly commit/push, lint, export

## Key relationships
- Clients:Tasks hub ↔ Vault:Task Data  
- Clients:Dashboards hub ↔ Vault:Dashboard Data  
- Vault:Dashboard Data ↔ Plugins Input → Plugins Output ↔ Clients:Dashboards hub  
- Vault (cfg & tasks) ↔ GitHub `origin/main`  
- Vault → Automations (CI, lint, export)

Open `docs/architecture/diagram.mmd` in a Mermaid viewer (Obsidian, VSCode Mermaid, GitHub).
