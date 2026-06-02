# I-821D Approval Calculator

A simple, anonymous calculator that estimates when an I-821D (DACA renewal) case might be approved, based on recent approval-date trends from [MyCasesHub](https://mycaseshub.com).

**Live calculator:** https://catchingexcalibur.github.io/approval_calculator/

> **Scope:** This tool only covers **I-821D (DACA renewal)** cases. It does not estimate timelines for I-90, I-485, N-400, or any other form type.

---

## What it does

The calculator has three modes:

1. **Current batch**: shows which range of submission dates USCIS is currently approving, and projects which dates come next over the following weeks.
2. **When will I be approved?**: given your Date of Submission, estimates which approval week your case is likely to fall in.
3. **When should I submit?**: given a card expiration date, suggests the latest submission date that should still result in approval before your card expires.

---

## A note on terms: DOS

Throughout the calculator, **DOS** means **Date of Submission**. Technically this is the date your case last had a status update before approval (the data field is called `last_before_approval_date`), which for most people is the date they submitted their renewal. Earlier versions of this project called this "LBA"; it has been renamed to DOS for clarity.

---

## How it works

USCIS doesn't approve I-821D cases in random order. Each week, they approve cases whose Date of Submission (DOS) falls inside one or more clusters of dates. The calculator tracks two things:

- **Where the main cluster currently is**: for the most recent week of data, the center of the largest group of submission dates being approved.
- **How fast that cluster moves forward**: how many days of submission dates USCIS gets through per calendar week.

For example: if USCIS is currently approving cases with a DOS around Nov 20, 2025, and the main cluster has been advancing about 6.5 days of submission dates per calendar week, then someone with a DOS of Jan 5, 2026 is 46 days "down the line," meaning roughly 7 calendar weeks before USCIS reaches their case.

That's the core model. Anchor on the current main cluster, project forward at the observed pace.

### Why "current wait time" math doesn't work

A common mistake is to take the latest reported wait time (e.g. "187 days") and add it to your submission date. That only works for people whose submission date is already at the front of the queue. If you submitted more recently, your wait will be longer, because the queue keeps growing.

The calculator avoids this by using queue position instead of average wait time.

---

## The methodology evolved (main-cluster model)

The calculator originally used a simpler model: USCIS was assumed to work a single, narrow batch of submission dates each week, advancing it forward at a steady rate. This held for roughly 10 weeks (February through April 2026), with the frontier advancing about 2.5 DOS-days per calendar week.

In May 2026, USCIS shifted to a **multi-cluster approval pattern**: instead of one batch, they began approving a large main cluster of cases alongside one or two smaller groups of newer cases in the same week. This broke the single-batch model, because the overall median submission date bounced around depending on the week's mix and sometimes appeared to move backwards, which is not physically meaningful.

The current model handles this by **anchoring on the center of the main cluster** (the largest concentration of approvals in a given week) rather than the overall median. The smaller secondary clusters are treated separately. This is more honest about where the bulk of approvals are actually happening.

Across the three most recent weeks, the main cluster advanced at a consistent pace:

| Approval week | Main cluster center (DOS) | Advance |
|---|---|---|
| May 11, 2026 | around Nov 7, 2025 | (baseline) |
| May 18, 2026 | around Nov 13, 2025 | +6 days |
| May 25, 2026 | around Nov 20, 2025 | +7 days |

That works out to about 6.5 DOS-days per calendar week, which is the slope the calculator currently uses.

---

## The model in detail

The model uses three numbers, all derived from the data:

| Constant | Meaning | Current value |
|---|---|---|
| `LATEST_BATCH_P50` | Main-cluster median DOS in the most recent week | 2025-11-20 |
| `FRONTIER_SLOPE` | DOS-days advanced per calendar week | +6.5 d/wk |
| `BATCH_WIDTH_DAYS` | Typical width of the approval cluster | 9 days |

For any submission date `S`, the model computes:

```
calendar_weeks_until_approval = (S − LATEST_BATCH_P50) / FRONTIER_SLOPE
estimated_approval_week       = LATEST_APPROVAL_WEEK + (calendar_weeks_until_approval × 7 days)
```

The "likely window" treats the cluster as a fixed 9-day-wide interval that slides forward at the frontier slope. The earliest possible approval is when the front edge of the cluster reaches the user's date; the latest is when the back edge sweeps past.

Approval dates that fall on a Saturday or Sunday roll forward to Monday, since USCIS approvals only happen on weekdays.

---

## Data source

All approval data is sourced from **[MyCasesHub](https://mycaseshub.com)** via weekly snapshots of I-821D approvals. The trend constants are computed from clean cases only:

- `success = True`
- `match_type = processing_to_approved`
- All three of `last_before_approval_date`, `approved_date`, and `days_last_to_approval` populated

Cases are deduplicated by case number across snapshots. Weeks with fewer than 20 clean cases are excluded from the trend fit.

The current model is built on **15 weeks of data covering roughly 7,800 I-821D approvals** (Feb 9 through May 25, 2026).

---

## Privacy

**No data is collected, logged, or transmitted.** The calculator runs entirely in your browser:

- No database, no analytics, no tracking
- No receipt numbers, A-numbers, names, or any personal information requested
- The only input required is a date
- All math happens locally in JavaScript on your device

If you want to verify this, the entire calculator is in `index.html` in this repo. Read it yourself.

---

## Files

This repo currently contains a single file:

| File | Purpose |
|---|---|
| `index.html` | The calculator, a self-contained HTML/CSS/JavaScript file |

The model constants are hardcoded into the `D` block near the top of `index.html`. To update the calculator with new data, edit those constants and commit.

---

## Limitations

This calculator is a planning tool, not a guarantee. It will not predict your approval date precisely, and the underlying assumptions can break in any of these ways:

- **USCIS changes pace or pattern**: they could speed up, slow down, change which clusters they work, or stop entirely. The shift to a multi-cluster pattern in May 2026 is a real example of this happening.
- **Holidays and shutdowns**: federal closures pause processing.
- **Case-specific factors**: RFEs, biometrics issues, or different processing tracks pull individual cases off the standard queue.
- **Sample size**: the main-cluster slope is based on a handful of recent weeks; estimates more than about 8 weeks into the future are particularly uncertain.
- **I-821D only**: the model is fit on DACA-renewal data and should not be used for other form types.

The model only knows what the recent past has looked like and assumes the near future will look similar. **Use this as one input among many when planning, not as a definitive answer.**

---

## Roadmap

The calculator is updated weekly as new MyCasesHub snapshots come in. Things being watched and considered as the dataset grows:

- Whether the multi-cluster pattern continues, and whether the secondary (newer-case) clusters grow large enough that the definition of the "main" cluster needs to change.
- Refining the frontier slope as more weeks of multi-cluster data accumulate.
- Confidence bands derived from observed week-over-week variance.

---

## Disclaimer

This calculator is not affiliated with USCIS, MyCasesHub, or any government agency. It is an independent project that uses publicly accessible approval data to produce statistical estimates. Estimates are based on observed trends and have no official standing. Approval timing depends on many factors outside the model's view.

**Do not make legally consequential decisions based solely on this calculator.** Consult an immigration attorney for case-specific guidance.

---

## Contributing

If you spot a bug or see the math doing something obviously wrong, open an issue. Suggestions welcome.

---

## Thanks

Thanks to everyone who's been following along on r/DACA and supported the project. This exists because official processing-time estimates aren't granular enough to plan around. Hopefully this fills a small piece of that gap.

- [@CatchingExcalibur](https://github.com/CatchingExcalibur)
