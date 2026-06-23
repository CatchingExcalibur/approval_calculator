# DACA Approval Estimator

A simple, anonymous tool that estimates when a DACA renewal (Form I-821D) might be approved, based on recent approval data sourced from [MyCasesHub](https://mycaseshub.com).

**Live tool:** https://catchingexcalibur.github.io/approval_calculator/

> **Scope:** This tool only covers I-821D (DACA renewal) cases. It does not estimate timelines for I-90, I-485, N-400, or any other form type.

---

## What it does

You enter the date you submitted your DACA renewal, and the tool gives you a rough estimate of when it might be approved, based on how USCIS has been approving similar cases recently.

That's it. One date in, an estimated approval window out. No accounts, no personal information, no tracking.

---

## How USCIS processing changed, and why this tool works the way it does

This tool has been rebuilt several times because USCIS kept changing how they process cases. Here is the short history, because it explains the current design.

**February through April 2026: one slow line.** For about ten weeks, USCIS worked through cases like a single-file line, approving them roughly in the order they were submitted and advancing about 2.5 days worth of submission dates per calendar week. The early versions of this tool modeled that as a slowly moving "frontier."

**Late April 2026: a split into two tracks.** On April 27, 2026, USCIS started a new, stronger background-check process. Cases whose biometrics were taken before that date had to be re-screened, which slowed them down. Cases submitted after that date were screened under the new system from the start, so they moved through quickly. This split the data into two separate groups, and the single-line model stopped working.

**June 2026: high-volume clearing of both tracks.** USCIS dramatically increased their approval volume to roughly 3,000+ cases per week, about four times the earlier pace. Some weeks they focus that capacity on recent applications, other weeks on the older backlog. Recent applications now get approved within roughly a month. The older backlog, which had been nearly stuck, is now being cleared quickly in large weekly waves.

The current tool reflects this with a **two-track model**.

---

## The two-track model

When you enter your submission date, the tool sorts you into one of two groups based on a cutoff around April 27, 2026.

**Fast track (submitted on or after about April 27, 2026).** Recent applications are being approved a roughly flat number of days after submission, currently about 25 to 55 days, assuming the background check is clear. The estimate is simply your submission date plus that window.

**Older group / backlog (submitted before late April 2026).** These cases went through the extended background-check process. They are a longer wait, currently around 150 to 205 days from submission, but this backlog is now being cleared quickly, so the wait has been compressing week to week.

**Edge zone (within about two weeks of the cutoff).** Cases submitted right around late April can fall on either side depending on when their background check was run, so the tool shows both possibilities.

The cutoff date and the wait ranges are derived from the most recent weekly MyCasesHub data and are updated as new data comes in.

---

## A note on the April 27 cutoff

The April 27, 2026 dividing line matches the date USCIS rolled out its enhanced vetting process. The interpretation that this cutoff is what splits cases into the fast and slow groups is the developer's best reading of the data and public policy news. It fits the observed pattern well, but USCIS has not officially confirmed that this specific date drives the split. Treat it as a well-supported guess, not a confirmed fact.

---

## Data source

All approval data is sourced from [MyCasesHub](https://mycaseshub.com) via weekly snapshots of I-821D approvals. The model constants are computed from clean cases only:

- `success = True`
- `match_type = processing_to_approved`
- All of `last_before_approval_date`, `approved_date`, and `days_last_to_approval` populated

Cases are deduplicated by case number across snapshots. The tool is built on roughly 18 weeks of data covering more than 10,000 I-821D approvals (February through June 2026).

In the data and earlier versions of this tool, the submission date is referred to as DOS (Date of Submission). Technically this is the date a case last had a status update before approval, which for most people is when they submitted their renewal.

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

The model constants are hardcoded in the `D` block near the top of `index.html`. To update the tool with new weekly data, edit those constants and commit.

---

## Follow the updates

Weekly updates are posted on the r/DACA subreddit, where new data is shared, changes to the tool are explained, and questions are answered. The full list of updates is linked from the `updates.html` page, and the latest update is always the best place to see the current state of the data.

Subreddit: https://www.reddit.com/r/DACA/

---

## Limitations

This is a planning tool, not a guarantee. It will not predict your approval date precisely, and the underlying assumptions can break:

- USCIS changes pace or approach, which has already happened several times in 2026
- Holidays and shutdowns pause processing
- Case-specific factors like RFEs, biometrics issues, or country of origin can pull individual cases off the typical pattern
- The current two-track pattern is only a few weeks old, so the numbers may shift as more data comes in

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
