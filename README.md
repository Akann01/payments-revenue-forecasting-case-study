Payments Revenue Forecasting & Variance Analysis
A Finance Analytics Case Study

> Read this first: This is a designed case study based on a simulated scenario, not a record of an engagement I ran for a real company. All numbers are labeled assumptions chosen to be realistic. The mechanisms are drawn from work I have actually done: optimizing multi-table SQL queries and validating outputs against expected results before deployment (John Deere internship), and building a self-service Power BI dashboard with automated daily refresh for an 8-9 member team (John Deere internship).

Author: Akanksha Mishra | Role in the scenario: Data Analyst, Business Finance (forecasting, variance analysis, automated reporting; no access to real company data)

## Scenario Assumptions

| Item | Assumption |
|---|---|
| Company type | Mid-size payments/fintech processor |
| Merchant categories | 4 (e-commerce, subscriptions, B2B, EMI) |
| History simulated | 18 months |
| Reporting cadence | Monthly, with automated refresh |
| Baseline | Manual spreadsheet reconciliation, ~3 days/month |
| Driver | Finance needs same-day visibility into which categories are missing forecast, not a 3-day-lagged report |

## 1. Executive Summary & Scale Metrics

Why now. Manual month-end variance reconciliation across merchant categories takes days, by which point the window to act on a revenue miss has mostly closed. The business needs a same-day, automated view of where actuals are diverging from forecast and why.

Mission. Replace manual spreadsheet reconciliation with an automated pipeline that forecasts monthly revenue per merchant category, flags variance against plan, and surfaces the top drivers without anyone pulling numbers by hand.

Scope of ownership. I owned the forecasting model, the SQL-based variance logic, and the reporting layer end to end, simulating the kind of ownership a Data Analyst on a Business Finance team would have.

North Star KPI: Same-day variance visibility for 100% of merchant categories, down from a 3-day manual cycle.

| Metric | Baseline | Target |
|---|---|---|
| Time to variance visibility | ~3 days (manual) | Same day (automated) |
| Categories flagged automatically at >10% deviation | 0 | 100% |
| Manual reconciliation effort | ~3 days/month | <2 hours/month (review only) |
| Forecast categories covered | 0 | 4/4 |

Mechanism in one line: a forecast-vs-actual table refreshed automatically, with SQL ranking categories by variance, replaced manual spreadsheet comparison so review time went to deciding what to do about a miss, not finding it.

## 2. Pipeline Architecture

Three stages, each with one clear output feeding the next.

| Stage | Output | Tooling |
|---|---|---|
| 1. Data Simulation | 18 months of transaction volume, revenue, fees, and refund rate per category, with seasonality and planted anomalies | Python, Pandas |
| 2. Forecasting | Next-month revenue forecast per category | Python, linear regression (chosen for interpretability over marginal accuracy) |
| 3. Variance Analysis & Reporting | Forecast-vs-actual variance ranked by category, auto-refreshing dashboard view | SQL, Power BI |

Design choice: an explainable model over a more accurate black-box one. A finance stakeholder needs to trust and act on the number same-day, not audit a model.

## 3. Variance Logic & Thresholds

| Rule | Definition |
|---|---|
| Green | Actual within 5% of forecast |
| Amber | Actual 5-10% below forecast |
| Red | Actual >10% below forecast, flagged for same-day review |
| Review trigger | Any category in Red for 2 consecutive months escalates to a root-cause note, not just a number |

SQL does the ranking and flagging; the dashboard surfaces only what crossed a threshold, so review time goes to the categories that matter, not to re-deriving the same table every month.

## 4. Pivot / Data Quality Scenario (designed scenario)

Situation. In month 11 of the simulated data, the B2B category shows a 22% revenue drop against forecast, large enough to trigger Red.

Detection. The automated variance query flags it the same day the actuals load, instead of surfacing in a month-end spreadsheet review.

Response

1. Checked the refund-rate and transaction-volume breakdown for B2B before assuming the forecast was simply wrong.
2. Found volume held steady but average transaction value dropped sharply, pointing to a mix shift rather than demand loss.
3. Re-ran the forecast with mix-adjusted inputs rather than retraining on the anomaly, to avoid the model learning a one-off as a trend.
4. Logged the finding as a one-line root-cause note alongside the flagged row, so the next reviewer doesn't re-investigate the same drop.

Lesson. A variance flag should come with a first-pass "why," not just a number — otherwise the automation saves time finding the problem but not diagnosing it.

## 5. Projected Results & Business Value

These are design targets for the scenario, not delivered results.

| Outcome | Target |
|---|---|
| Variance visibility | Same day, down from ~3 days |
| Manual reconciliation effort removed | ~3 days/month, replaced by <2 hours of review |
| Categories with automated forecast + variance tracking | 4/4 |
| Root-cause context attached to Red flags | 100% |
| Reusable assets | Forecasting script, variance SQL logic, dashboard template |

## What I'd Do Differently at Larger Scale

- Validate the forecasting model against a longer history before trusting mix-shift vs. true-demand-loss calls.
- Add a lightweight anomaly-detection pass before the threshold rule, so single-category spikes don't need a human to notice the pattern first.
- Build the root-cause note field into the dashboard from day one, instead of as a follow-up fix.
