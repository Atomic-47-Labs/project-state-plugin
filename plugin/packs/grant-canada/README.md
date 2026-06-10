# grant-canada

**Version 1.0.0** · **Requires** project-state core >= 3.0

Canadian grant submission lifecycle for project-state. Adds a pre-award `grant-state/` facility and three skills covering 19 major Canadian programs. On award, bridges seamlessly to a sibling `project-state/` execution facility.

## What this pack adds

**3 new skills:**
- `grant-state` — memory layer for `grant-state/` submission facilities (mirrors `project-state`'s role for the pre-award phase)
- `grant-scaffolder` — scaffold a submission facility; match playbook; seed sections, gates, budget; award handoff to `project-state/`
- `grant-ingestor` — drain inbox of program docs; produce strategy pass; eligibility verdict; overnight harvest from Gmail/Slack

**4 pack profiles** (configure existing project-state skills post-award):
- `phase-gate` — execution lifecycle with grant-specific gate conditions (carry-forward verification, data retention)
- `funder-reporting` — progress reports, quarterly claims, and final reports per Canadian program
- `review-meeting` — Grant Advisory Committee and Steering Committee meeting patterns
- `external-comms` — OCAP, GBA+, bilingual, TTO routing, stacking disclosure rules

**19 Canadian playbooks** (built into `grant-scaffolder` and `grant-ingestor`):
Tri-Council (NSERC/SSHRC/CIHR), IRAP, SIF, PIC, CFI JELF, Mitacs Accelerate, NGen, SCALE.AI, Genome Canada, PacifiCan, FedDev Ontario, FedNor, CED Québec, ACOA, CanNor, SR&ED, agnostic-core fallback.

**12 compliance gate types:**
OCAP®, GBA+, bilingual, TTO routing, stacking, IP declaration, Indigenous engagement, environmental, cost-share, ethics-REB, biosafety, data sovereignty.

## Quick start

```
# 1. Scaffold a new submission
/grant-scaffolder
→ Program: NSERC Alliance
→ Deadline: 2026-09-15
→ ...

# 2. Drop the program guide into the inbox
# Copy grant guide PDF + eligibility doc + partner materials into:
# grant-state/documents/inbox/

# 3. Triage and produce strategy pass
/grant-ingestor triage
/grant-ingestor strategy

# 4. Each morning during the submission sprint
/grant-ingestor harvest   # pull overnight signals
# then work on sections: /grant-state + editor

# 5. On award day
/grant-scaffolder
→ "we won the grant"
→ Records award, freezes grant-state/, spawns project-state/
→ Carries forward: people, IP, milestones, gate snapshots

# 6. Continue in project-state/
/project-onboarding   # completes execution facility setup
```

## Two-facility model

```
my-project/
├── grant-state/      ← pre-award (this pack)
│   ├── manifest.yaml
│   ├── sections/     ← narrative drafts
│   ├── gates/        ← compliance gates
│   ├── letters/      ← support letters
│   └── logs/
│
└── project-state/    ← post-award (core project-state)
    ├── manifest.yaml
    ├── milestones/   ← re-baselined from submission
    ├── people/       ← consortium members carried forward
    └── logs/
```

The two facilities are siblings. On award, `grant-scaffolder` bridges them — the submission's work becomes the foundation of the five-year project.

## Compatibility

- Compatible with `sred-canada` pack (complementary — SR&ED covers tax credits; grant-canada covers grant submissions)
- Compatible with `pic-pcais` pack (the `pic` playbook in grant-canada handles PIC submissions; `pic-pcais` handles PIC execution reporting — use both together for full PIC project lifecycle)

## Authors

Atomic 47 Labs Inc. (Kelowna, Canada) — https://atomic47.co

