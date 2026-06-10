# invest-like consensus record

A **tamper-evident, append-only daily record** of which US-listed stocks pass
5 or more of the 7 investor frameworks scored by
[invest-like.com](https://invest-like.com) (Buffett, Graham, Fisher, Lynch,
Greenblatt, Munger, Terry Smith).

## Why this exists

invest-like publishes a track record: the cohort of stocks passing **all 7**
frameworks beat the S&P 500 by a median **+73.8 percentage points over 5
years** (n = 47). The standard, fair objection to any such claim is
*"self-published backtest - how do I know the picks weren't backfilled?"*

This repository is the answer. Every day, an automated GitHub Actions
workflow (in this repo, code visible to everyone) fetches the **public**
consensus API and commits the complete 5+/7 cohort here. From that point on:

- **Every pick is timestamped** by the git commit history before its future
  performance is knowable.
- **Nothing can be silently rewritten.** Each snapshot embeds the SHA-256 of
  the previous snapshot (`prev_snapshot_sha256`), forming a hash chain.
  Rewriting history would break the chain for anyone who has ever cloned or
  forked this repo.
- **Anyone can replicate it.** The data source is a public, keyless
  endpoint - you can fetch it yourself and compare.

## Layout

```
snapshots/YYYY-MM-DD.json   one file per day, never modified after the day ends
latest.json                 copy of the most recent snapshot
```

Each snapshot contains: `recorded_at_utc`, `prev_snapshot_sha256`, `source`
(the exact URL fetched), and `data` (the API response: every stock with its
bull count, bullish frameworks, composite score, top grade, sector, halal
status).

## Verify it yourself

```bash
# 1. Does today's data match what the API serves right now?
curl -s "https://invest-like.com/api/public/consensus?min=5&limit=300" | jq .cohort_size

# 2. Is the hash chain intact?
sha256sum snapshots/<yesterday>.json   # compare to prev_snapshot_sha256 in today's file

# 3. Did any pick get backfilled? Check when a ticker first appeared:
git log --diff-filter=A --follow -- snapshots/ | head
git log -S '"TICKER"' --oneline -- snapshots/
```

## Methodology

- Scoring rubrics, pillar weights, and deal-breaker caps:
  [invest-like.com/methodology](https://invest-like.com/methodology/)
- Working paper (permanent DOI):
  [10.5281/zenodo.20393518](https://doi.org/10.5281/zenodo.20393518)
- Full annual report with cohort returns:
  [State of Value Investing 2026](https://invest-like.com/reports/state-of-value-investing-2026/)

A framework counts a stock as a "bull" at score >= 60 (B+ or better) against
that investor's documented rubric. `bulls` is how many of the 7 agree.

## License

Data: **CC BY 4.0** - reuse, chart, or republish freely with attribution to
invest-like.com. Educational data only; nothing here is investment advice.
