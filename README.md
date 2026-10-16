# smarttube-lab

Independent, evidence-first analysis of the [SmartTube](https://github.com/yuliskov/SmartTube)
open-source project — plus the tooling that produces it.

This is a lab notebook: not a fork, not a review, not a request to anyone. Everything here comes
from public data (the GitHub API and public diffs), every number is reproducible with the
commands cited in each document, and corrections are published with dates.

## Method

1. **Measure first.** No claim leaves this lab without a script run against the live API and the
   raw output archived.
2. **Verify with diffs.** Similarity scores nominate candidates; reading the actual diffs decides.
   Our own first-pass figure ("~22% of the open PR queue is duplicate work") did not survive that
   step — the published map says so, in the open.
3. **Publish claim + falsification.** Each finding lists what would refute it; corrections get
   their own entry.

## Findings

- [Open-PR overlap map (2026-10-15)](findings/2026-10-overlap-map.md) — triage map of the open PR
  queue of SmartTube: which PRs collide, which genuinely compete, and which similarity flags are
  false positives. (49 open PRs → 5 groups; after reading the diffs: 1 genuine duplicate pair.)

## Tooling

- [find-dupes](https://github.com/josejim/find-dupes) — flags overlapping open PRs in any GitHub
  repository. The SmartTube run is its case study.

## Conduct & disclaimers

- **Independent.** Not affiliated with the SmartTube project or its maintainer. No maintainer
  privileges were used; everything derives from public data.
- **Similarity ≠ duplication.** Overlap flags are candidates for human review, never verdicts.
  (This map's own headline number was corrected by exactly that step.)
- **No upstream assets are redistributed here.** The only contribution posted upstream from this
  account is a Spanish translation PR (merged 2026-10-15).
