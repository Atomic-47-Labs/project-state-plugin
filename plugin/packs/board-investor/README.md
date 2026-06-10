# Board / Investor Pack

For PE/VC-backed startup projects that need to report to a board and/or investors on a regular cadence.

## What you get

- `profiles/review-meeting.yaml` — Board Meeting lifecycle, monthly or bi-monthly cadence, board roster, board pack assembly.
- `profiles/funder-reporting.yaml` — Monthly investor update + KPI snapshot, with the investor mailing list as the recipient.
- Templates for board pack and monthly investor update.
- `reporting-matrix-defaults.yaml` — seeds matrix entries for `board` and `investors` stakeholder groups.

## Loading

```yaml
project:
  packs_loaded: [board-investor]
```

Pairs naturally with `client-services` (a startup doing customer engagements) or with `pic-pcais` (a startup that's also funded by PIC).

## Maturity

Starter. Tune the board pack template and KPI list to match what your board actually wants to see each month.
