# Agile Engineering Defaults Pack

For engineering teams running Scrum/Kanban with release trains. This pack does not assume a funder or customer; load it alongside other packs for those.

## What you get

- `profiles/review-meeting.yaml` — Sprint retrospective lifecycle, sprint cadence (default 2 weeks, configurable), team attendees, retro template (Start/Stop/Continue).
- `profiles/phase-gate.yaml` — Sprint phases as the project lifecycle (Discovery → Build-loops → Hardening → Release), with sprint-end as the gate.
- `reporting-matrix-defaults.yaml` — seeds matrix entries for `team` and `engineering-leadership` stakeholder groups.

## Loading

```yaml
project:
  packs_loaded: [agile-default]
```

Or alongside others (e.g. `agile-default + client-services` for an agile team doing client work; `agile-default + board-investor` for an agile startup).
