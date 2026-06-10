---
name: project-kanban
description: "Start and open the project-state local kanban dashboard — a Next.js app at project-state/kanban/ that runs on port 3355. Serves five views: Kanban (milestones as cards in status columns with harvest surface chips), Dashboard (activity timeline, milestone burndown, risk heatmap), Inventory (milestones, risks, decisions, documents — each with inline markdown viewer), Reports (baseline bundles for download, weekly and adhoc reports viewable inline), and Milestone detail. Trigger on '/project-kanban', 'open the kanban', 'start the kanban', 'open the dashboard', 'launch the project UI', 'show me the milestones', or any request to open, start, or check the project-state web interface."
---

# project-kanban — the local dashboard launcher

## Purpose

`project-kanban` starts the project-state local reporting application and opens it in the browser. The app reads `project-state/` directly — no database, no sync step. Every page load is a fresh read from the flat-file substrate.

The app runs on **http://localhost:3355** and has five pages:

| Page            | URL                  | Question answered                                                  |
|-----------------|----------------------|--------------------------------------------------------------------|
| Kanban          | /kanban              | What's the status of every milestone? What's moving or at risk?   |
| Dashboard       | /dashboard           | Activity over time, milestone burndown, risk distribution.         |
| Inventory       | /inventory           | All milestones, risks, decisions, and documents — searchable.      |
| Reports         | /reports             | Baseline bundles (download) + weekly/adhoc reports (view inline).  |
| Milestone detail | /milestone/[id]     | Full milestone card: deliverables, activity, harvest signals.      |

## Location

```
project-state/kanban/           ← Next.js 15 app (relative to project root)
├── src/app/                     ← pages and API routes
├── src/components/              ← kanban-app, dashboard-app, inventory-app,
│                                   reports-app, milestone-detail-app, nav,
│                                   markdown-viewer
└── package.json                 ← "dev": "next dev -p 3355"
```

The kanban resolves `project-state/` by walking up from `cwd` — no `PROJECT_STATE_DIR` env var required when launched from inside `project-state/kanban/`.

## Subcommands

| Subcommand | What it does                                              |
|------------|-----------------------------------------------------------|
| (none)     | Default: start server if needed, open browser             |
| `start`    | Start dev server only (don't open browser)                |
| `stop`     | Kill the process on port 3355                             |
| `status`   | Report whether server is running + project-state summary  |
| `restart`  | Stop then start                                           |
| `open`     | Open browser (assumes server already running)             |

## Execution steps

### Step 1 — Find the kanban directory

The kanban lives at `project-state/kanban/` relative to the project root. Locate it by:

1. Check `$PROJECT_STATE_DIR/../kanban/` if env is set.
2. Otherwise walk up from `cwd` to find `project-state/kanban/`.
3. If not found, initialise it from the packaged starter. The starter ships from the project-state package either expanded (`templates/kanban/`) or, in uploaded/zip distributions, as a compressed archive (`templates/kanban.tgz`) — the app's App Router uses bracketed dynamic-route folders (`milestone/[id]`) that some zip uploaders reject, so packaged builds tar it. Use whichever is present:
   ```bash
   mkdir -p project-state/kanban
   if [ -f templates/kanban.tgz ]; then
     tar xzf templates/kanban.tgz -C project-state/kanban
   elif [ -d templates/kanban ]; then
     cp -R templates/kanban/. project-state/kanban/
   else
     echo "kanban starter not found — run /project-scaffolder or check you're in the project directory." >&2; exit 1
   fi
   ```

### Step 2 — Check node_modules

```bash
ls project-state/kanban/node_modules 2>/dev/null | head -1
```

If empty or missing:
```bash
cd project-state/kanban && npm install
```

### Step 3 — Check if server is already running

```bash
lsof -ti tcp:3355 | head -1
```

- PID returned → server is up. Skip to Step 5.
- Empty → proceed to Step 4.

### Step 4 — Start the dev server

```bash
cd project-state/kanban && npm run dev > /tmp/project-kanban.log 2>&1 &
```

Wait up to 15 seconds for port 3355:

```bash
for i in $(seq 1 15); do
  lsof -ti tcp:3355 > /dev/null 2>&1 && break
  sleep 1
done
```

If port never opens, report the error and show the last 20 lines of `/tmp/project-kanban.log`.

### Step 5 — Open browser (unless `start` subcommand)

```bash
open http://localhost:3355/kanban
```

On Linux use `xdg-open`. If neither is available, print the URL.

### Step 6 — Print project-state summary

Read `project-state/state.json` and `project-state/manifest.yaml` and print a brief status:

```
✓ project-kanban running at http://localhost:3355/kanban

Project: <manifest.project.name>
  Phase:       <state.current_phase>
  Milestones:  <counters.milestones> total
  Risks:       <counters.risks> open
  Decisions:   <counters.decisions> logged
  Harvest:
    scsiwyg    <last cursor or "never">
    gmail      <last cursor or "never">
    slack      <last cursor or "never">
    gdocs      <last cursor or "never">
```

Pull counters from `state.json:counters`, phase from `state.json:current_phase`, harvest dates from `state.json:harvest_cursors`.

## Stop subcommand

```bash
PID=$(lsof -ti tcp:3355)
if [ -n "$PID" ]; then
  kill $PID
  echo "Stopped project-kanban (PID $PID)"
else
  echo "project-kanban is not running"
fi
```

## Status subcommand

Check port 3355 and print the project-state summary regardless.

## Restart subcommand

Run Stop then Start in sequence.

## Error handling

| Error                                   | Action                                                                             |
|-----------------------------------------|------------------------------------------------------------------------------------|
| `project-state/kanban/` not found      | Tell the user the kanban hasn't been initialised and suggest `/project-scaffolder`.|
| `node_modules` missing                  | Run `npm install` inside `project-state/kanban/` before starting.                 |
| Port 3355 in use by something else      | Report which process owns it (`lsof -i tcp:3355`) and ask before killing.          |
| Server starts but never responds        | Show last 20 lines of `/tmp/project-kanban.log`.                                   |
| `manifest.yaml` missing                 | Skip project summary; report that the project hasn't been scaffolded yet.          |

## Notes

- The kanban reads `project-state/` on every page load — no restart needed after running `/project-harvester`, `/project-milestone-manager`, or any other skill.
- Harvest surface chips (SL/GM/GD/SC) on milestone cards are populated from `project-state/documents/inbox/` — run `/project-harvester` to refresh them.
- Baseline report bundles in the Reports tab are served from `project-state/reports/baseline/`.
- Milestone detail pages live at `/milestone/{id}` — navigate from any milestone card's "Detail →" link.
