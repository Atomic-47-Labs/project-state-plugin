# Client Services Pack

For consulting/client-engagement projects where the project's "external party" is a paying customer rather than a funder.

## What you get

- `profiles/funder-reporting.yaml` — monthly customer invoicing, milestone-based billing entries, percent-complete-of-contract-value, invoice cover-email template.
- `profiles/review-meeting.yaml` — Quarterly Business Review (QBR) lifecycle, customer attendees, QBR pack template, monthly status updates as a cadence option.
- `profiles/external-comms.yaml` — customer-confidentiality review (NDA enforcement, sensitive-data scrubbing), customer-approval-required for any external mention of the engagement.
- `profiles/archive.yaml` — engagement wrap workflow: customer final deliverable handover, customer signoff on completion, post-engagement support window setup, customer documentation freeze.
- Templates for invoice cover-email, QBR pack, engagement-final-deliverable.
- `reporting-matrix-defaults.yaml` — seeds matrix entries for `customer.<id>` stakeholder group.

## Loading

```yaml
project:
  packs_loaded: [client-services]
```

Or alongside another funder/grant pack:

```yaml
project:
  packs_loaded: [pic-pcais, client-services]
```

When both are loaded, the funder pack handles funder-side reporting (PIC quarterly claim, SC) and the client-services pack handles customer-side reporting (monthly invoice, QBR). Both contribute reporting matrix entries; the orchestrator dispatches each on its own cadence.

## Maturity

This is a **starter pack** — the shape is right but you'll likely want to tune the invoicing template, QBR agenda, and signoff gates to match your specific client contract. Treat the profiles as well-commented starting points, not finished spec.
