# DACA Approval Estimator

A simple, anonymous tool that estimates when a DACA renewal (Form I-821D) might be approved, based on recent approval data sourced from [MyCasesHub](https://mycaseshub.com).

**Live tool:** https://catchingexcalibur.github.io/approval_calculator/

**Background and weekly updates:** [updates.html](updates.html)

> **Scope:** This tool only covers I-821D (DACA renewal) cases. It does not estimate timelines for I-90, I-485, N-400, or any other form type.

---

## What it does

You enter the date you submitted your DACA renewal, and the tool gives you a rough estimate of when it might be approved, based on how USCIS has been approving similar cases recently.

That is it. One date in, an estimated approval window out. No accounts, no personal information, no tracking.

---

## How USCIS processing changed, and why this tool works the way it does

This tool has been rebuilt several times because USCIS kept changing how they process cases. Here is the short history, because it explains the current design.

**February through April 2026: one slow line.** For about ten weeks, USCIS worked through cases like a single-file line, approving them roughly in the order they were submitted and advancing about 2.5 days of submission dates per calendar week.

**Late April 2026: a split into two tracks.** On April 27, 2026, USCIS started a new, stronger background-check process. Cases whose biometrics were taken before that date had to be re-screened, which slowed them down, while cases submitted after that date were screened under the new system from the start and moved quickly. This split the data into two separate groups.

**June 2026: a massive catch-up.** USCIS increased approvals to several thousand per week, peaking at over 8,800 in a single week. The backlog collapsed. The two tracks merged into one smooth pattern.

**July 2026: a steady state.** With most of the backlog cleared, weekly volume settled to around 3,000. Each week is now the last of the older cases finishing their extended review, plus new applications approved in about three weeks.

The current tool reflects this final stage with a **single catch-up model**.

---

## The single catch-up model

Since USCIS has essentially caught up, the estimate comes down to two simple questions.

First, has the application been in the system longer than the minimum processing time, which is currently about 25 days? If yes, the application is in the current wave, and cases like it are being approved around now. If the application is more recent than that, the estimate is the submission date plus that minimum time.

The data behind this is unusually clean. When you plot wait time against submission date, it forms a nearly perfect straight line with a slope of about 1.0. In plain terms, almost everyone getting approved in a given week is landing on the same few days, regardless of when they applied. The only thing that changes a person's wait is how long ago they submitted.

### The dynamic anchor

Rather than freezing an approval date at the time of each weekly update, the tool computes the estimate from the current day every time it runs. The anchor (the point that caught-up cases are measured against) is simply today, plus a few days of headroom. This keeps the estimate accurate between weekly data refreshes instead of drifting stale.

One side effect worth knowing: for an application already past the minimum wait, the estimated date moves forward day by day as the calendar advances. This is not the wait growing. It is the tool continuing to say "around now" and following the calendar until the approval actually lands.

### Weekends

USCIS does not approve cases on weekends, so any estimated date that falls on a Saturday or Sunday is rolled forward to the following Monday.

---

## The model constants

All constants live in the `D` block near the top of `index.html` and are refreshed from the latest weekly MyCasesHub snapshot.

| Constant | Meaning | Current value |
|---|---|---|
| `LATEST_APPROVAL_WEEK` | Monday of the most recent approval week | 2026-07-13 |
| `MIN_WAIT` | Minimum processing time for new applications | 25 days |
| `ANCHOR_LAG` | Days of headroom added to today for the anchor | 4 days |
| `BAND_LOW` / `BAND_HIGH` | Window shown before and after the midpoint | 10 / 21 days |

To update the tool with new data, edit those constants and commit.

---

## A note on the April 27 cutoff

The two-track split that appeared in spring 2026 lined up with the April 27, 2026 date that USCIS rolled out enhanced vetting. The interpretation that this cutoff drove the split, with earlier applications effectively reviewed twice, is the developer's best reading of the data and public policy news. It fits the observed pattern well, but USCIS has not officially confirmed it. Treat it as a well-supported explanation, not a confirmed fact.

---

## Data source

All approval data is sourced from [MyCasesHub](https://mycaseshub.com) via weekly snapshots of I-821D approvals. The model constants are computed from clean cases only:

- `success = True`
- `match_type = processing_to_approved`
- All of `last_before_approval_date`, `approved_date`, and `days_last_to_approval` populated

Cases are deduplicated by case number across snapshots. The tool is built on roughly 22 weeks of data covering more than 29,000 I-821D approvals (February through July 2026).

In the data and earlier versions of this tool, the submission date is sometimes referred to as DOS (Date of Submission). Technically this is the date a case last had a status update before approval, which for most people is when they submitted their renewal.

---

## Privacy

No data is collected, logged, or transmitted. The tool runs entirely in your browser:

- No database, no analytics, no tracking
- No receipt numbers, A-numbers, names, or any personal information requested
- The only input is a date
- All calculations happen locally in JavaScript on your device

If you want to verify this, the entire tool is in `index.html` in this repo. Read it yourself.

---

## Files

| File | Purpose |
|---|---|
| `index.html` | The estimator, a self-contained HTML/CSS/JavaScript file |
| `updates.html` | An about/background page with an intro and links to the weekly Reddit updates |

---

## Follow the updates

Weekly updates are posted on the r/DACA subreddit, where new data is shared, changes to the tool are explained, and questions are answered. Every update includes the full, unabridged list of approved submission dates for the most recent weeks, so anyone can check the numbers directly. The complete list of updates is linked from the `updates.html` page.

Subreddit: https://www.reddit.com/r/DACA/

---

## Limitations

This is a planning tool, not a guarantee. It will not predict your approval date precisely, and the underlying assumptions can break:

- USCIS changes pace or approach, which has already happened several times in 2026
- Holidays and shutdowns pause processing
- Case-specific factors like RFEs, biometrics issues, or country of origin can pull individual cases off the typical pattern
- Older applications (submitted before late April 2026) can wait longer than the model suggests, since some are still finishing the extended background-check review

The model only knows what the recent past has looked like and assumes the near future will look similar. Use this as one input when planning, not as a definitive answer. For advice about your own case, talk to a licensed immigration attorney.

---

## Disclaimer

This tool is not affiliated with USCIS, MyCasesHub, or any government agency. It is an independent project that uses publicly accessible approval data to produce statistical estimates. Estimates are based on observed trends and have no official standing.

Do not make legally consequential decisions based solely on this tool. Consult an immigration attorney for case-specific guidance.

---

## Contributing

If you spot a bug or see the math doing something obviously wrong, open an issue. Suggestions are welcome.

---

## Thanks

Thanks to everyone who has followed along on r/DACA, shared feedback, and asked hard questions. This exists because official processing-time estimates are not detailed enough to plan around. Hopefully it helps fill a small piece of that gap.

- [@CatchingExcalibur](https://github.com/CatchingExcalibur)
