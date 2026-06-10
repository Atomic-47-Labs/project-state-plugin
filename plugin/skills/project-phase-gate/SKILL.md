---
name: project-phase-gate
description: Manage lifecycle phase transitions for any project. v2.0 supports user-defined phase sets via presets in templates/phase-presets/ — grant-default (LOI→Approval→Planning→Execution→Closeout→Archive), agile-default (Discovery→Build-loops→Hardening→Release), waterfall-default, client-engagement-default (Discovery→Proposal→Engagement→Wrap), open-source-default (Incubation→Active→Maintained→Archived), or custom. Active pack can override gate-in/gate-out criteria per phase. Enforces required artifacts; refuses to transition if gate artifacts missing. Use whenever the user says 'what phase are we in', 'can we move to execution', 'what's blocking the gate', 'transition to the next phase', 'gate status', 'gate checklist'.
---

# Project Phase Gate (v2.0 — user-defined phases)

Manages the lifecycle phase transitions of a project. Each phase has a gate-in (what must be true to enter) and a gate-out (what must be true to leave). The skill refuses transitions when gate artifacts are missing.

In v2.0, phase definitions are no longer hard-coded. They come from a preset (`templates/phase-presets/<preset-name>.yaml`) selected in the manifest, with optional overrides from active pack profiles.

## Available presets (ship in v2.0)

- **`grant-default`** — LOI → Approval → Planning → Execution → Closeout → Archive. Reproduces v1.x lifecycle. Used by grant projects (PIC, NSERC, NIH, EU Horizon, etc.).
- **`agile-default`** — Discovery → Build-loops (recurring) → Hardening → Release. For engineering teams running Scrum/Kanban with release trains.
- **`waterfall-default`** — Requirements → Design → Build → Test → Deploy → Maintain. For traditional waterfall projects.
- **`client-engagement-default`** — Discovery → Proposal → Engagement → Wrap. For consulting/client-services work.
- **`open-source-default`** — Incubation → Active → Maintained → Archived. For community-governed projects.
- **Custom** — write your own preset YAML; reference it from `manifest.yaml` as `phases.preset: "your-preset"`.

## Pack overrides

A pack can ship a `phase-gate.yaml` profile that adds or modifies gate criteria for a preset. Example: the PIC pack's profile augments `grant-default` with PIC-specific gate-in/gate-out criteria (e.g. "MPA signed by all parties" as planning gate-out). Loading the PIC pack adds those checks; loading no pack uses the bare preset.

## What it owns

- Reading the current phase from `state.json`
- Enforcing gate-in and gate-out checklists per the active preset + pack overrides
- Refusing transitions with clear errors when checklists are incomplete
- Writing transition events to `logs/activity.ndjson`
- Updating `state.json` on successful transitions

## What it does not own

- Defining what the phases are — that's the preset
- Defining gate criteria — those are the preset + pack overrides
- Doing the work to satisfy a gate — that's other skills + humans

## Migration from v1.x

The skill name is unchanged. The hard-coded six-phase grant lifecycle moves to `templates/phase-presets/grant-default.yaml` (verbatim). Existing projects' `state.json` and phase records are unchanged. The PIC-specific gate criteria move to `packs/pic-pcais/profiles/phase-gate.yaml`.

A v1.x project loading the PIC pack and selecting `phases.preset: "grant-default"` gets identical behavior to v1.x. A non-grant project picks a different preset (or writes a custom one) and the same skill manages it.
