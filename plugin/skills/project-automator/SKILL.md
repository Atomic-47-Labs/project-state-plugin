---
name: project-automator
description: "Compile the project's reporting-matrix.yaml into a machine-readable automation/schedule.yaml that a scheduling host (keep-state, cron, or any external scheduler) can consume. Reads every matrix entry, classifies it as cadence (cron) or event-driven (hook), applies the configured time window, reasons through current milestone and phase state to activate or suppress achievement-based triggers, then writes project-state/automation/schedule.yaml. Modes: plan (preview), generate (write), update (re-diff), status (last-run + enabled count). Does NOT register crons itself — the host app reads schedule.yaml. Trigger: /project-automator"
---

# project-automator

## Purpose

The reporting matrix is the source of truth for *what* runs. This skill is the compiler
that turns the matrix into a structured schedule any host can execute — without the user
having to translate cadences by hand or re-register jobs every time the matrix changes.

Two tracks:

| Track | Source | Compiled to |
|---|---|---|
| **Cadence** | Matrix entries with `kind: weekly \| monthly \| quarterly \| annual \| sprint-aligned \| bi-weekly` | Cron expressions placed inside the configured time window |
| **Achievement** | Matrix entries with `kind: ad-hoc \| post-event \| on-publish` + current milestone/phase state | Event-hook definitions + optional one-shot activations |

Output: `project-state/automation/schedule.yaml` — a single derived file. The reporting
matrix remains the source; re-running this skill regenerates the schedule from scratch.

---

## Invocation

```
/project-automator               → plan mode (preview, no writes)
/project-automator generate      → write automation/schedule.yaml
/project-automator update        → diff current schedule vs. matrix; patch changed entries
/project-automator status        → show last-run, enabled jobs, pending achievement triggers
/project-automator --window 23:00-05:00   → override time window
/project-automator --tz America/Vancouver → override timezone
```

---

## Step 0 — Locate and load state

1. Walk up from cwd to find `project-state/manifest.yaml`. Fail fast if not found.
2. Read `project-state/reporting-matrix.yaml` → `entries[]`.
3. Read `project-state/state.json` → current phase, health, milestone pointers.
4. Read `project-state/manifest.yaml` → `id`, `timezone` (if set under `automation.timezone`
   or `project.timezone`). If timezone is absent from both, **prompt**: "What timezone should
   scheduled jobs run in? (e.g. America/Vancouver)" before proceeding.
5. Read window from args, then from `manifest.yaml:automation.window`, then default `23:00–05:00`.

---

## Step 1 — Classify entries

For each entry in `entries[]`:

### Cadence entries (→ cron jobs)

| `cadence.kind` | Logic |
|---|---|
| `weekly` | Emit one cron per week on `cadence.day`. Place it at `window.start`. |
| `bi-weekly` | Emit one cron on `cadence.day` (odd/even week via `%2`). |
| `monthly` | Emit one cron. If `day_of_month: last-business-day` → last weekday of month; `first-monday` → first Monday; else nth day. Place at `window.start + 30min` to stagger. |
| `quarterly` | Emit one cron per quarter boundary aligned to `cadence.alignment`. Place at `window.start + 60min`. |
| `annual` | Emit one cron on `cadence.due_month`. Place at `window.start + 90min`. |
| `sprint-aligned` | Emit one cron on sprint-end day (read from `state.json:sprint_calendar`). |

**Time-window placement rule:** distribute jobs evenly across the window so they do not
all start at 23:00. Spacing = `window_minutes / job_count` (minimum 5 min between jobs).
Assign each job a slot in order of priority (URGENT deadlines first, then weekly, monthly,
quarterly, annual). Never schedule past `window.end`.

**lead_time offsets:** subtract `lead_time_days` or `lead_time_hours` from the natural
due date when computing the cron fire time. E.g. a monthly report due on the 1st with
`lead_time_days: 2` fires on the 29th/30th.

### Event-driven entries (→ hooks)

| `cadence.kind` | Emitted hook `on:` field |
|---|---|
| `ad-hoc` | `cadence.on_event` verbatim (e.g. `milestone.completed`, `phase.transition`) |
| `post-event` | `cadence.on` (e.g. `sc-meeting.held`) with `within_business_days` preserved |
| `on-publish` | `cadence.trigger` (e.g. `documents/published/`) |

For `ad-hoc` entries, also **inspect current state** to determine if the trigger condition
is already satisfied (the achievement already happened but the report wasn't run):
- `on_event: milestone.completed` → check `milestones/` for any `status: completed` with
  no corresponding activity-log `report.generated` entry since the completion. If found,
  set `activation: immediate` in the hook — the host should fire it on next available slot.
- `on_event: phase.transition` → check if the current phase is newer than the last
  `phase-gate.transition` log event. If so, set `activation: immediate`.

---

## Step 2 — Build schedule.yaml

```yaml
schema_version: 1
manifest_kind: automation_schedule
generated_from: project-state/reporting-matrix.yaml
generated_at: <ISO-8601 UTC>
project: <manifest.id>

window:
  start: "23:00"
  end:   "05:00"
  timezone: "America/Vancouver"

jobs:
  # ── Cadence (cron) ───────────────────────────────────────────────
  - id: <entry.id>
    type: cron
    cron: "<computed cron expression>"    # standard 5-field
    skill: "project-orchestrator tick --entry <entry.id>"
    label: "<entry.report>"
    generator: <entry.generator>
    surfaces: <entry.surface split into array>
    enabled: true                          # false if entry has enabled: false
    source_entry: <entry.id>
    next_due: "<ISO-8601 date>"            # informational; host may ignore

  # ── Achievement (event hooks) ────────────────────────────────────
  - id: <entry.id>
    type: event
    on: "<on_event or on or trigger>"
    within_business_days: <N>              # post-event entries only
    skill: "project-orchestrator tick --entry <entry.id>"
    label: "<entry.report>"
    generator: <entry.generator>
    surfaces: <entry.surface split into array>
    enabled: true
    activation: deferred                   # or: immediate (already triggered, not yet run)
    source_entry: <entry.id>
```

One job per matrix entry. Disabled matrix entries (`enabled: false`) emit `enabled: false`
jobs — the host sees them but skips execution.

---

## Step 3 — Output by mode

### `plan` (default)
Print the compiled job list to the terminal. **Do not write any files.** Show:

```
# Automation plan — <project-id> — <date>

Window: 23:00–05:00 America/Vancouver
Source: project-state/reporting-matrix.yaml (<N> entries)

## Cron jobs (<N>)
  <time>  <entry.id>  →  <generator>  [<surface>]
  ...

## Event hooks (<N>)
  on:<event>  <entry.id>  →  <generator>  [<surface>]
  ...
  ⚡ <entry.id>  activation=immediate  (trigger already met — fires on next slot)

To write: /project-automator generate
```

### `generate`
Write `project-state/automation/schedule.yaml`. Confirm with the user before writing if
any `activation: immediate` jobs are present ("These N jobs have already met their trigger
condition and will fire on the next scheduler run. Proceed?"). Append one
`automator.generate` event to `logs/activity.ndjson`.

### `update`
Read the existing `automation/schedule.yaml`. Diff it against the freshly-compiled plan.
Print a change summary (added / removed / changed entries). Ask "Apply these changes?"
before writing. Append `automator.update` event to activity log.

### `status`
Read `automation/schedule.yaml` and tail the activity log for `automator.*` events.
Report: last generated, enabled job count, immediate-activation count, last tick timestamp
(if the host writes it back to `state.json:automation_last_tick`).

---

## Automation config in manifest.yaml

The skill reads (and the scaffolder may seed) this optional block:

```yaml
# project-state/manifest.yaml
automation:
  window:
    start: "23:00"
    end: "05:00"
  timezone: "America/Vancouver"
  enabled: true           # master switch; if false, all emitted jobs have enabled: false
```

If the block is absent, the skill uses defaults and prompts for timezone on first run.
After prompting, it offers to write the block back: "Save these settings to manifest.yaml
for future runs?"

---

## What this skill does NOT do

- Does not register crons. The host (keep-state or otherwise) reads `schedule.yaml` and
  registers them. This skill is the compiler, not the executor.
- Does not call generators directly. That is the orchestrator's `tick` routine.
- Does not modify the reporting matrix. The matrix is the source of truth; this skill
  only reads it.
- Does not manage the 11pm–5am window enforcement at runtime. The host enforces that via
  the cron expressions this skill emits.
- Does not send, post, or draft anything. Read + compute + write one file.

---

## Integration

**Triggered by the user** (`/project-automator generate`) when the matrix changes or a
new project is set up.

**Triggered by `project-scaffolder`** after seeding the reporting matrix from packs —
it calls `project-automator generate` as the final step so the schedule is ready
immediately.

**Read by keep-state (or any host)** on its polling interval. The host is responsible for
detecting when `schedule.yaml` changes (via mtime or git hash) and re-loading it.

**The kanban Schedule view** (`/schedule`) renders `schedule.yaml` alongside the reporting
matrix so a human can see the computed crons, toggle individual jobs (`enabled: false`),
and see `activation: immediate` warnings — without having to read YAML directly.
