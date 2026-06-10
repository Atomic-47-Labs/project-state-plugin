---
name: project-archive
description: Project closeout and archival drive. Generic core handles final reports, lessons summary assembly, archive directory creation, audit-trail finalization. Funder/customer-specific closeout items (PIC final reports, FTE confirmation, holdback release, MPA close) come from the active pack's archive profile. PIC pack ships the v1.x closeout flow. Client-services pack ships customer-final-deliverable + sunset workflow. Use whenever the user says 'close the project', 'closeout', 'final report', 'wrap up', 'submit final reports', 'archive the project', 'ready to close', 'holdback release', 'project end', or any request related to the closeout phase.
---

# Project Archive (v2.0 — generic core + pack-driven closeout)

Drives the closeout phase: final reports, lessons-learned summary, IP final reporting, financial reconciliation, archive directory creation, audit-trail finalization.

In v2.0 the skill splits into a **generic closeout core** and **pack-driven closeout items**:

**Generic core** (always runs):
- Final lessons-learned summary assembled from `lessons-learned/`
- All milestones marked complete or explicitly archived-incomplete
- Audit trail finalized (last `activity.ndjson` entry timestamped)
- Archive directory created at `project-state/archive-<closeout-date>/`
- Final state.json snapshot
- Closeout README pointing future readers at the archived state

**Pack-driven items** (from active pack's `archive.yaml` profile):
- PIC pack: PIC final report per consortium member (~90-day pre-close template distribution, drafts by close date, finalized 30 days post-close), IP final reporting workflow, FTE confirmation, holdback release tracking, Annual Questionnaire close-out.
- Client-services pack: customer final deliverable handover, customer signoff on completion, post-engagement support window setup, customer documentation freeze.
- Open-source pack: archive notice on README, contributor recognition publication, governance handoff or sunset declaration.

## What it owns

- Closeout phase orchestration (gate-in: project end date reached; gate-out: all required artifacts complete)
- Final-report drafting per the active pack's template(s)
- Lessons-learned summary
- Archive directory + final state snapshot
- Closeout audit trail

## What it does not own

- Submitting final reports — drafts only, PL signs off
- Releasing holdback (or any payment) — humans coordinate that with the funder
- Defining what "closed" means — the gate-out criteria are in the phase manifest + pack profile

## Migration from v1.x

The skill name is unchanged. The PIC-specific closeout items (final report template, FTE confirmation, holdback release, ~90-day pre-close window, 30-day post-close finalization) move from hard-coded to `packs/pic-pcais/profiles/archive.yaml`. Existing closeout work-in-progress is unchanged.

For non-grant projects the generic closeout core still runs (lessons summary, archive directory, final snapshot) — pack-driven items contribute additional steps.
