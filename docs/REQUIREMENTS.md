# Runway Room: Product Requirements

Version 1 requirements, written 12 November 2026.

## What we are building

Runway Room answers one question for an early-stage founder: how many months of cash do we have, and how does that change if we hire, grow or raise?

- **Problem.** Founders track cash in an ad hoc spreadsheet. Answering "what if we hire two people in January?" means rebuilding formulas each time.
- **Primary user.** A founder of a company with under 30 people who has a bank or ledger export and no finance hire.
- **Secondary user.** A founder's office or finance associate preparing the monthly investor update.
- **The job.** Upload transactions, see burn and runway in under a minute, test three kinds of change, and leave with a drafted update.
- **Goal.** A live, public demo that a founder can use with sample data in 30 seconds, with maths that can be defended line by line.

## Version 1 scope

Version 1 has ten features and no login, no database and no saved accounts.

| # | In version 1 | Build day |
| --- | --- | --- |
| 1 | Upload a CSV of transactions, with a column-mapping step | 3 |
| 2 | "Try with sample data" button using a fictional startup | 3 |
| 3 | Automatic categorisation by keyword rules, editable per row | 3 |
| 4 | Monthly cash in, cash out, gross burn, net burn and runway | 4 |
| 5 | Dashboard: runway headline, monthly burn chart, cash projection chart | 5 |
| 6 | "Default alive or default dead" verdict on the dashboard | 5 |
| 7 | "Start fundraising by" date on the dashboard | 5 |
| 8 | Scenarios: new hire, revenue growth, fundraise, compared against the baseline | 6 |
| 9 | Reverse questions: how much to raise for a target runway, and the earliest affordable hire | 7 |
| 10 | Drafted investor update from the numbers, with a plain template as fallback | 9 |

| Deliberately out | Why |
| --- | --- |
| Login and saved workspaces | Adds a database and a week of work; nothing in the demo needs it |
| Bank connections | Needs paid, regulated integrations |
| Multiple currencies | One currency, rupees, shown in lakh and crore format |
| Milestone boards, KPI trackers, team priorities | Different products. This one does cash |
| Accounting accuracy (accruals, GST, depreciation) | This is a cash view, and the app says so |

Planned after launch, in this order: PDF export, shareable scenario links, a US dollar display option.

## The calculations

Runway is found by projecting cash month by month until it goes below zero, not by a single division. That one method handles the baseline and every scenario.

| Term | Definition |
| --- | --- |
| Operating cash in | Sum of inflows in a month, excluding the Funding category |
| Cash out | Sum of all outflows in a month |
| Gross burn | Cash out for the month |
| Net burn | Cash out minus operating cash in. Negative net burn means the month was cash-flow positive |
| Trailing average | Mean of the last 3 complete calendar months. The user can switch to 1 or 6. A partial current month is ignored |
| Current cash | Entered by the user. Not derived from the file, because exports rarely start at account opening |
| Baseline projection | Each future month: cash = previous cash + average operating cash in − average cash out |
| Runway | Full months before cash goes below zero, plus the fraction: cash left at the start of the final month ÷ that month's net burn |
| Cash-out date | Today plus the runway, shown as month and year |
| No finite runway | If cash never goes below zero within 36 months, show "36+ months" |

### Scenarios

| Scenario | Inputs | Effect on the projection |
| --- | --- | --- |
| New hire | Monthly cost, start month | Adds the cost to cash out from that month onward |
| Revenue growth | Monthly growth rate | Operating cash in for month t = average operating cash in × (1 + rate)^t |
| Fundraise | Amount, month received | Adds the amount to cash in that month. Not counted as operating cash in |

Scenarios can be combined. Each is always shown beside the baseline.

### Verdict, deadline and reverse questions

| Output | Definition |
| --- | --- |
| Default alive | With the user's revenue growth rate applied, monthly net burn reaches zero or below before cash goes below zero, within 36 months |
| Default dead | Any other case, including zero growth with a positive net burn |
| Start fundraising by | Cash-out date minus the raise lead time. Lead time defaults to 6 months and the user can change it |
| Raise needed for a target runway | The smallest amount, received now, that keeps projected cash at or above zero through the target month |
| Earliest affordable hire | The earliest start month for which total runway from today stays at or above the user's target |

### Worked checks

Current cash ₹60,00,000. Net burn in the last three months: ₹7,50,000, ₹8,00,000, ₹8,50,000. No growth.

- Average net burn = ₹8,00,000. Baseline runway = 60,00,000 ÷ 8,00,000 = **7.5 months**.
- Add a hire costing ₹1,50,000 a month from month 3. Months 1 and 2 burn ₹8,00,000 each, leaving ₹44,00,000. From month 3 the burn is ₹9,50,000. After month 6, cash is ₹6,00,000. Runway = 6 + 6,00,000 ÷ 9,50,000 = **6.6 months**.
- Verdict: **default dead**, because burn never falls.
- Start fundraising by: 7.5 − 6 = **1.5 months from today**.
- Raise needed for 18 months: 18 × 8,00,000 − 60,00,000 = **₹84,00,000**.
- Earliest hire at ₹1,50,000 a month that keeps 7 months of runway: **month 6** (runway 7.1 months). Starting in month 5 gives 6.9, which misses the target.

These become the first unit tests.

## Data in, screens and privacy

The app accepts one CSV and never sends transactions to a server. Parsing and every calculation happen in the visitor's browser.

### The file

| Column | Required | Notes |
| --- | --- | --- |
| Date | Yes | The user picks the format; default is DD/MM/YYYY |
| Description | Yes | Used for categorisation |
| Amount | Yes, in one of two forms | One signed column (positive in, negative out), or separate debit and credit columns as Indian bank exports usually have |
| Category | No | If present, it overrides the keyword rules |

Categories: Revenue, Funding, Payroll, Rent, Software, Marketing, Taxes, Other. Rows the rules cannot place go to Other and are flagged for review.

The sample dataset is a fictional company with 9 months of invented transactions. No real company's data is used anywhere.

### Screens

1. **Start.** One sentence, an upload area, and "Try with sample data".
2. **Map and review.** Match columns, set current cash, fix categories.
3. **Dashboard.** Months of runway as the headline, cash-out date, verdict, burn chart, projection chart.
4. **Scenarios.** Three controls beside the projection, baseline always visible.
5. **Investor update.** The drafted text, editable, with a copy button.
6. **How the numbers work.** The definitions above in plain language.

Design: cream background, navy text, gold only on the runway figure. Fraunces for the headline number, Inter for everything else. It must work on a phone.

### Privacy and security

- Transactions stay in the browser and vanish when the tab closes. The start screen says this.
- The investor-update draft sends only monthly totals and scenario results to the AI service, never individual transactions or descriptions. The screen says this before the user clicks.
- The AI key lives in a server-side environment variable and never appears in the repo or the browser.
- Every result carries the line: "A cash-based estimate, not accounting or financial advice."

## Done means

Version 1 is finished when every line below is true on the live URL.

- [ ] A first-time visitor reaches a runway number from sample data in under 30 seconds
- [ ] The worked checks return 7.5 and 6.6 months in the app and in the unit tests
- [ ] The verdict and reverse-question checks return default dead, 1.5 months, ₹84,00,000 and month 6
- [ ] A company whose growth reaches break-even before cash-out shows "default alive"
- [ ] The app's monthly figures match the Excel model for the sample dataset, to the rupee
- [ ] A CSV with debit and credit columns imports correctly
- [ ] A file with a missing column, a blank row or a badly formatted date shows a clear message, not a crash
- [ ] A company with positive cash flow shows "36+ months", not an error or a negative number
- [ ] Changing any scenario input updates the chart without a page reload
- [ ] When the AI limit is reached, the investor update falls back to the template
- [ ] Every screen is usable on a phone
- [ ] Three people outside the project have used it and their top issues are fixed
- [ ] No API key appears anywhere in the GitHub history
- [ ] The README carries the copyright line and the repo has no open-source licence
