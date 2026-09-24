# Project Report: Lending Club Default Risk Analysis

**Author:** Vasu Vachhani ([@vvachhani28-ux](https://github.com/vvachhani28-ux))
**Repository:** [lending-club-default-analysis](https://github.com/vvachhani28-ux/lending-club-default-analysis)

---

## 1. Business Problem

Lending Club is a peer-to-peer lending platform. For every loan application, the company must decide whether to approve it. Two types of errors are costly:

- **Approving a loan that later defaults** → loss of the principal (and any unpaid interest).
- **Rejecting an applicant who would have repaid** → lost business/interest income.

This project analyzes historical loan-level data to understand **which applicant and loan characteristics are associated with a higher probability of default**, using only variables that would realistically be known **at the time of application** — not variables that reflect what happened *after* the loan was issued (repayment history, recoveries, last payment date, etc.), since those would not be available for a real approval decision.

## 2. Dataset

- **Source:** Lending Club loan-level dataset, loans issued 2007–2011.
- **Scope after cleaning:** 37,544 loans with a final status of `Fully Paid` or `Charged Off`. Loans still marked `Current` at the time of data extraction were excluded since their eventual outcome is unknown.
- **Target variable:** `default_flag` — 1 if `Charged Off`, 0 if `Fully Paid`.
- **Overall default rate:** 14.4% (5,399 of 37,544 loans).

### 2.1 Data Cleaning Steps

1. Dropped columns with more than 90% missing values.
2. Dropped `desc` (free text) and `mths_since_last_delinq` (high missingness, low signal).
3. Converted `int_rate` from a percentage string to a numeric value.
4. Cleaned `emp_length` into a numeric `emp_length_years` field.
5. Removed **behavior variables** — fields only knowable after a loan is issued (e.g. `total_pymnt`, `recoveries`, `last_pymnt_d`, `delinq_2yrs`, `revol_util`) — to prevent data leakage.
6. Removed identifier/geographic columns not useful for aggregate risk analysis (`title`, `url`, `zip_code`, `addr_state`).
7. Filtered to closed loans only (`Fully Paid` / `Charged Off`) and derived the binary target.
8. Parsed `issue_d` into year/month for time-based analysis.
9. Created business-meaningful bins (bands) for continuous variables — loan amount, interest rate, DTI, installment, annual income, employment length — while preserving the original numeric columns for flexible analysis.

## 3. Methodology

- **Univariate analysis:** default rate distribution across each categorical/binned variable independently (grade, purpose, term, home ownership, verification status, income band, interest rate band, DTI band).
- **Bivariate/segmented analysis:** default rate by `purpose` cross-tabbed with `grade`, to see whether purpose-level risk holds consistently across credit grades or is concentrated in specific grade/purpose combinations.
- **Time-based analysis:** loan volume and default rate trended by issue year, to check whether risk was stable as the platform scaled.
- **Excel pivot tables:** a lightweight cross-tab view (loan status × grade, term, and purpose) as a sanity check against the Python analysis.
- **Power BI dashboard:** an interactive layer on top of the same cleaned dataset, allowing filtering by year, grade, and term across all breakdowns simultaneously.

## 4. Findings

### 4.1 Loan Grade — the strongest predictor

| Grade | Default Rate |
|---|---|
| A | 5.8% |
| B | 11.9% |
| C | 16.8% |
| D | 21.8% |
| E | 26.7% |
| F | 32.6% |
| G | 33.6% |

Default rate increases almost monotonically from A to G — a nearly 6x difference between the safest and riskiest grades. This is unsurprising (grade is Lending Club's own risk assessment), but it confirms the platform's internal grading was directionally sound over this period, and it's the single most useful variable for a quick risk read.

### 4.2 Interest Rate Band

| Band | Default Rate |
|---|---|
| Low (≤10%) | 6.4% |
| Medium (≤15%) | 14.5% |
| High (>15%) | 26.0% |

Since interest rate is largely a function of grade, this closely mirrors the grade finding — but it's a useful independent check, since rate is a continuous variable rather than a categorical assignment.

### 4.3 Loan Term

| Term | Default Rate |
|---|---|
| 36 months | 10.9% |
| 60 months | 25.1% |

Longer-term loans default at more than double the rate of shorter-term loans. This may partly reflect that riskier borrowers self-select (or are steered) into longer terms, and partly reflect that a 60-month repayment horizon gives more time for a borrower's financial situation to deteriorate.

### 4.4 Loan Purpose

| Purpose | Default Rate |
|---|---|
| small_business | 27.0% |
| renewable_energy | 19.1% |
| house | 16.7% |
| educational | 16.4% |
| other | 16.1% |
| medical | 15.4% |
| moving | 15.2% |
| debt_consolidation | 15.2% |
| vacation | 14.4% |
| home_improvement | 11.8% |
| car | 10.7% |
| credit_card | 10.4% |
| major_purchase | 10.1% |
| wedding | 10.1% |

`small_business` loans stand out sharply — nearly double the average default rate, and the highest of any purpose. This tracks with intuition: business income is more volatile than personal income, and small business failure rates are historically high.

### 4.5 Purpose × Grade Interaction

Cross-tabbing purpose against grade (see the dashboard heatmap) shows that `small_business` remains elevated **at every grade level**, not just in the lower grades — meaning purpose adds risk information beyond what grade alone captures. This is the strongest case for using purpose as an independent input to underwriting, not just a byproduct of who tends to apply for what.

### 4.6 Income and DTI

- **Annual income band:** default rate decreases from ~16.6% (Low income) to ~10.8–10.9% (High/Very High income) — a real but moderate effect.
- **DTI band:** default rate increases from ~12.5% (Low DTI) to ~16.4% (High DTI) — also real, but smaller in magnitude than grade, rate, or term effects.

These variables are useful for fine-tuning risk within a grade, but are weaker standalone predictors than grade, interest rate, or term.

### 4.7 Time Trend

| Year | Loans Issued | Default Rate |
|---|---|---|
| 2007 | 251 | 17.9% |
| 2008 | 1,562 | 15.8% |
| 2009 | 4,716 | 12.6% |
| 2010 | 11,214 | 12.6% |
| 2011 | 19,801 | 15.7% |

Default rate dipped through 2009–2010 (likely partly a function of loan seasoning — 2010/2011 loans had less time to default by the data extraction date) before rising again in 2011, alongside a near-doubling of loan volume. This pattern is worth flagging rather than over-interpreting, since younger loan vintages mechanically show lower *observed* default rates simply because they've had less time to default.

## 5. Recommendations

1. **Tighten underwriting or re-price Grade E/F/G loans.** These carry default rates 4–6x that of Grade A. If continuing to originate them, ensure the interest rate charged adequately compensates for the loss rate.
2. **Add purpose-specific scrutiny for `small_business` and `renewable_energy` loans.** The elevated risk persists across grades, suggesting purpose carries independent signal — consider additional documentation requirements or exposure caps for these categories.
3. **Reassess the availability of 60-month terms**, particularly for mid-to-low grades, given the compounding effect of long term + weak grade.
4. **Treat vintage effects carefully when monitoring risk.** Since younger loans haven't had time to default, comparing raw default rates across issue years can be misleading — a cohort/survival-based view would give a fairer year-over-year comparison.
5. **Use income and DTI as pricing refinements, not gating criteria.** Their effect size is real but modest — better suited to adjusting rate within a grade than to accepting/rejecting outright.

## 6. Limitations

- This is an **observational, descriptive analysis** — it identifies association, not causation. A borrower's grade, for instance, is itself derived partly from the same underlying risk factors being studied.
- **Survivorship/vintage bias**: loans issued more recently (2011) have had less time to default than 2007 loans, which can understate their "true" eventual default rate.
- No predictive model (e.g. logistic regression, gradient boosting) was built in this phase — this is EDA-driven, aimed at identifying the strongest candidate features for a future predictive model.

## 7. Possible Next Steps

- Build a logistic regression or tree-based classification model using the cleaned features, and compare model-driven feature importance against the univariate findings here.
- Adjust default rates for loan age/vintage to get a fairer time-trend comparison.
- Extend the Excel and Power BI views with the DTI and income breakdowns for full parity with the notebook.
