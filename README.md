# I-821D Approval Calculator

A simple, anonymous calculator that estimates when an I-821D (DACA renewal) case might be approved, based on recent approval-date trends from [MyCasesHub](https://mycaseshub.com).

**Live calculator:** https://catchingexcalibur.github.io/approval_calculator/

> **Scope:** This tool only covers **I-821D (DACA renewal)** cases. It does not estimate timelines for I-90, I-485, N-400, or any other form type.

---

## What it does

The calculator has three modes:

1. **Current batch** — Shows which range of submission dates USCIS is currently approving, and projects which batch is up next over the following weeks.
2. **When will I be approved?** — Given your submission date, estimates which approval week your case is likely to fall in.
3. **When should I submit?** — Given a card expiration date, suggests the latest submission date that should still result in approval before your card expires.

---

## How it works

USCIS doesn't approve I-821D cases in random order. Each week, they approve cases whose **submission date** (the date your case last had a status update before approval — what the data calls `last_before_approval_date`, or "LBA") falls inside a small window of dates. The calculator tracks two things:

- **Where that window currently is** — for the most recent week of data, what submission dates are being approved.
- **How fast that window moves forward** — how many days of submission dates USCIS gets through per calendar week.

For example: if USCIS is currently approving cases with a submission date around Nov 20, 2025, and the window has been advancing about 2.5 days of submission dates per calendar week, then someone with a submission date of Jan 5, 2026 is 46 days "down the line" — meaning roughly 18 calendar weeks before USCIS reaches their case.

That's the whole model. Anchor on the current frontier, project forward at the observed pace.

### Why "current wait time" math doesn't work

A common mistake is to take the latest reported wait time (e.g. "153 days") and add it to your submission date. That only works for people whose submission date is already at the front of the queue. If you submitted more recently, your wait will be longer — because the queue is growing faster than USCIS is moving through it.

The calculator avoids this by using queue position instead of average wait time.

---

## The model in detail

The model uses three numbers, all derived from the data:

| Constant | Meaning | Current value |
|---|---|---|
| `LATEST_BATCH_P50` | Median submission date being approved in the most recent week | 2025-11-20 |
| `FRONTIER_SLOPE` | LBA-days advanced per calendar week (linear regression on weekly medians) | +2.52 d/wk |
| `BATCH_WIDTH_DAYS` | Median width of the approval batch (p75 − p25) across all weeks | 9 days |

For any submission date `S`, the model computes:

```
calendar_weeks_until_approval = (S − LATEST_BATCH_P50) / FRONTIER_SLOPE
estimated_approval_week       = LATEST_APPROVAL_WEEK + (calendar_weeks_until_approval × 7 days)
```

The "likely window" is computed by treating the batch as a fixed 9-day-wide interval that slides forward at the frontier slope. The earliest possible approval is when the front edge of the batch reaches the user's date; the latest is when the back edge sweeps past.

Approval dates that fall on a Saturday or Sunday roll forward to Monday, since USCIS approvals only happen on weekdays.

### Why a single slope and fixed width?

Earlier versions tried fitting separate slopes for the early edge, center, and late edge of the batch. With only ~10 weeks of data, those independent slopes absorbed too much noise — at one point the model predicted the batch window would invert (back edge passing front edge), which is mathematically impossible. The current single-slope, fixed-width model is the simplest honest approach for the data available. As the dataset grows, more sophisticated models can be tested against it.

---

## Data source

All approval data is sourced from **[MyCasesHub](https://mycaseshub.com)** via weekly snapshots of I-821D approvals. The trend constants in the calculator are computed from clean cases only:

- `success = True`
- `match_type = processing_to_approved`
- All three of `last_before_approval_date`, `approved_date`, and `days_last_to_approval` populated

Cases are deduplicated by case number across snapshots. Weeks with fewer than 20 clean cases are excluded from the trend fit (noise threshold).

The current model was fit on **10 weeks of data covering 6,295 I-821D approvals** (Feb 9 – Apr 24, 2026).

---

## Privacy

**No data is collected, logged, or transmitted.** The calculator runs entirely in your browser:

- No database, no analytics, no tracking
- No receipt numbers, A-numbers, names, or any personal information requested
- The only input required is a date
- All math happens locally in JavaScript on your device

If you want to verify this, the entire calculator is in `index.html` in this repo — read it yourself.

---

## Files

This repo currently contains a single file:

| File | Purpose |
|---|---|
| `index.html` | The calculator — a self-contained HTML/CSS/JavaScript file |

The model constants are hardcoded into the `D` block near the top of `index.html`. To update the calculator with new trend data, edit those constants and commit.

The data analysis pipeline (CSV ingestion, regression fitting, etc.) is run separately and isn't part of this repo. If there's interest, I may add it later.

---

## Limitations

This calculator is a planning tool, not a guarantee. It will not predict your approval date precisely, and the underlying assumptions can break in any of these ways:

- **USCIS changes pace** — they could speed up, slow down, or stop entirely
- **Holidays and shutdowns** — federal closures pause processing
- **Case-specific factors** — RFEs, biometrics issues, or different processing tracks pull individual cases off the standard queue
- **Sample size** — 10 weeks of data is a small foundation; estimates more than ~8 weeks into the future are particularly uncertain
- **I-821D only** — the model is fit on DACA-renewal data and should not be used for other form types

The model has no idea about any of these things. It only knows what the recent past has looked like and assumes the near future will look similar.

**Use this as one input among many when planning, not as a definitive answer.**

---

## Roadmap

The calculator will be updated weekly as new MyCasesHub snapshots come in. Improvements being considered as the dataset grows:

- More sophisticated trend models once 15+ weeks of data are available
- Detection and handling of structural breaks (e.g. if USCIS changes processing rate sharply)
- Confidence bands derived from observed week-over-week variance
- Exposing a recent-weeks slope as an alternate forecast for users who want a more conservative estimate

---

## Disclaimer

This calculator is not affiliated with USCIS, MyCasesHub, or any government agency. It is an independent project that uses publicly accessible approval data to produce statistical estimates. Estimates are based on observed trends and have no official standing. Approval timing depends on many factors outside the model's view.

**Do not make legally consequential decisions based solely on this calculator.** Consult an immigration attorney for case-specific guidance.

---

## Contributing

If you spot a bug or see the math doing something obviously wrong, open an issue. Suggestions welcome.

---

## Thanks

Thanks to everyone who's followed along on r/DACA and supported the project. This exists because official processing-time estimates aren't granular enough to plan around. Hopefully this fills a small piece of that gap.

— [@CatchingExcalibur](https://github.com/CatchingExcalibur)
