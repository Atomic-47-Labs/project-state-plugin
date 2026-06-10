# Worksona RUNBOOK: Leadership

*Leadership here is not a slogan — it's a **command you can run.***

> **Lead like you ship.** This runbook turns leadership principles into executable commands that guide how work gets mapped, built, measured, adopted, and connected. Each section provides a one-line command, a short rationale, and a repeatable loop **(advise → critique → build → analyze → learn)** so leaders can produce visible outcomes—not just plans. **Use it to align multi-disciplinary teams around fewer, better themes; prove progress with demos and data; and make interoperability and adoptability defaults rather than afterthoughts.**

## /demonstrate-not-decorate

**Description:** ***Ship proof, not polish.*** Favor the Smallest Valuable Slice (≤7 days) and tangible demos over polish. Require a 90-sec demo script.

**Runbook:** Advise on demo options → Critique to block polish theater → Build 7-day demo plan → Analyze user-visible signals → Learn from demo cycle.

```
/demonstrate-not-decorate --bias "Favor SVS" --advise "Propose 2-3 demo candidates, select 1" --critique "Block polish theater, enforce tangible value" --build "Create 7-day demo plan with telemetry" --analyze "Measure user-visible signals" --learn "Capture learning and refine cycle"
```

---

## /map-before-make

**Description:** ***Draw the terrain, then march.*** Always produce shared maps (journeys, workflows, interfaces) before building.

**Runbook:** Advise with diagrams → Critique missing interfaces → Build thin slices from map → Analyze integration contracts → Learn and update maps.

```
/map-before-make --bias "Always map before build" --advise "Output user journey + workflow diagrams" --critique "Flag missing interfaces or ownership" --build "Break map into thin backlog slices" --analyze "Validate integration contracts" --learn "Update maps and spread discipline"
```

---

## /drive-through-documentation

**Description:** ***Docs are a control surface.*** Treat documentation as a leadership instrument and living asset.

**Runbook:** Advise doc structures → Critique discoverability → Build repo scaffolds + CI → Analyze doc freshness → Learn via CI policy updates.

```
/drive-through-documentation --bias "Docs as leadership tool" --advise "Propose README/DECISIONS/RUNBOOK/CHANGELOG" --critique "Review discoverability and freshness" --build "Set up scaffolds and CI for docs" --analyze "Run doc freshness check" --learn "Update CI policies and simplify"
```

---

## /orchestrate-by-theme

**Description:** ***Fewer, bigger, better.*** Align under fewer, bigger, better themes that compound across teams.

**Runbook:** Advise 2–3 themes → Critique dilution/conflict → Build theme hub + cadence → Analyze integration/focus → Learn by updating theme narrative.

```
/orchestrate-by-theme --bias "Align under fewer, bigger themes" --advise "Define 2-3 themes and initiative mapping" --critique "Flag dilution/conflict" --build "Publish theme hub and review cadence" --analyze "Assess integration and focus" --learn "Update theme narrative and backlog"
```

---

## /enforce-measurement

**Description:** ***Evidence or it didn't happen.*** Require evidence at every loop; avoid anecdotes. Define north star, counters, and leading/lagging metrics.

**Runbook:** Advise metric tree → Critique Goodhart's risks → Build trackers + dashboards → Analyze signals vs targets → Learn to refine metrics.

```
/enforce-measurement --bias "Require evidence, avoid anecdotes" --advise "Produce metric tree and plan" --critique "Probe Goodhart's Law risks" --build "Implement trackers, dashboards, alerts" --analyze "Report signals vs targets" --learn "Refine metrics and align incentives"
```

---

## /champion-adoptability

**Description:** ***Adoption is the product.*** Make success equal ease of adoption, time-to-value, and reduced friction.

**Runbook:** Advise adoption paths → Critique onboarding TTV → Build quickstarts/templates → Analyze adoption frictions → Learn by simplifying flows.

```
/champion-adoptability --bias "Ease of adoption is success" --advise "Define adoption paths, quickstart, FAQ" --critique "Mystery-shop onboarding, measure TTV" --build "Create quickstart, templates, samples" --analyze "Report adoption frictions" --learn "Simplify flows, update docs and training"
```

---

## /mandate-interoperability

**Description:** ***Open by default.*** Default to open, discoverable interfaces; prevent silos.

**Runbook:** Advise interop matrix → Critique lock-in/version risks → Build adapters, schemas → Analyze compatibility → Learn by updating interop policy.

```
/mandate-interoperability --bias "Default to open interfaces" --advise "Produce interoperability matrix" --critique "Probe for lock-in and versioning risks" --build "Ship adapters, schemas, catalogs" --analyze "Run compatibility tests" --learn "Update interop policy and catalogs"
```

---

## How To Use the Decision Log

### 1. When to use it

Anytime you run a leadership command (e.g. /map-before-make, /enforce-measurement) and a decision is made.

Especially for choices with tradeoffs, dependencies, or future review points.

### 2. What to capture

Copy this template each time:

```
DECISION: <What was decided>
WHY: <Evidence/Tradeoffs>
ALTERNATIVES: <Considered>
OWNER: <Name>
DATE: <YYYY-MM-DD>
NEXT REVIEW: <YYYY-MM-DD>
```

**DECISION** → The choice made (clear, one sentence).

**WHY** → The reasoning and evidence (why we chose this).

**ALTERNATIVES** → Options that were considered but not chosen.

**OWNER** → Who is accountable for the decision.

**DATE** → When the decision was made.

**NEXT REVIEW** → When to revisit (to check if it's still valid).
