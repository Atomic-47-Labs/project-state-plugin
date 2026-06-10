# project-state — Claude Code plugin

A generic operational substrate for multi-stakeholder projects: **34 skills** that
make routine reporting a byproduct of normal work — milestones, objectives/KPIs,
phase gates, document curation, status reports, funder/grant compliance, and a
local kanban — configured by swappable **compliance packs**.

This repo is the **public distribution marketplace** for the plugin. (Development
happens in a separate private monorepo.)

## Install

```
/plugin marketplace add Atomic-47-Labs/project-state-plugin
/plugin install project-state@project-state-plugin
```

Then start a project:

```
/project-state:project-scaffolder
```

Skills auto-discover and namespace under `project-state:` — e.g.
`/project-state:project-milestone-manager`, `/project-state:project-orchestrator`,
`/project-state:project-kanban`. Claude also invokes them automatically by their
descriptions.

## Layout

```
.claude-plugin/marketplace.json   # marketplace manifest
plugin/                           # the plugin
├── .claude-plugin/plugin.json
├── skills/    (34)
├── packs/     (7 compliance packs)
└── templates/ (scaffolder seeds; kanban/website ship as .tgz)
```

## License

MIT
