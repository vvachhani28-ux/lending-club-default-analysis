# Lending Club Default Risk Analysis

**Which loans are most likely to default, and what should a lender do about it?**
An end-to-end data analytics project on the Lending Club consumer loan dataset (2007–2011) — covering Excel pivot analysis, Python/Jupyter EDA, and an interactive Power BI dashboard.

---

## 📌 Business Problem

Lending Club is a peer-to-peer lending platform that had to decide, for every loan application, whether to approve it. Approving a risky borrower leads to a loss of the principal; rejecting a safe borrower loses potential interest income. This project analyzes 37,544 closed loans (Fully Paid or Charged Off) to identify which applicant and loan characteristics are the strongest predictors of default — using only information that would have been available **at the time of loan approval** (no post-approval repayment behavior).

## 🔑 Key Findings

- **Overall default rate: 14.4%** across all closed loans.
- **Loan grade is the strongest single predictor** — default rate rises almost linearly from **5.8% (Grade A)** to **33.6% (Grade G)**.
- **Interest rate band tells the same story from a different angle**: Low-rate loans default at **6.4%**, High-rate loans at **26.0%** — a >4x difference.
- **Loan term matters a lot**: 60-month loans default at **25.1%** vs. **10.9%** for 36-month loans.
- **Loan purpose is a meaningful risk signal**: `small_business` loans default at **27.0%** — nearly double the overall rate — while `wedding` and `major_purchase` loans default at just **~10.1%**.
- **Income and DTI move risk in the expected direction**: higher income bands and lower debt-to-income bands both correlate with lower default rates, though the effect is smaller than grade or interest rate.
- **Risk was not static over time**: default rates dipped in 2009–2010 before climbing back up by 2011, alongside a large increase in loan volume — worth investigating whether underwriting standards loosened as the platform scaled.

## 💡 Recommendations

1. **Tighten underwriting on Grade E/F/G loans**, or price them with materially higher interest rates to compensate for a default rate 4–6x that of Grade A.
2. **Apply extra scrutiny to `small_business` and `renewable_energy` purpose loans** — both carry default rates well above average and may warrant additional documentation or lower approval limits.
3. **Reconsider 60-month term availability** for higher-risk grades — the combination of long term + low grade compounds risk substantially (see the Purpose × Grade heatmap in the dashboard).
4. **Monitor vintage-level risk, not just point-in-time risk** — the 2011 uptick in default rate alongside volume growth suggests risk controls should scale with origination volume, not stay fixed.
5. **Use income and DTI as secondary filters, not primary ones** — they shift default rates by only a few percentage points compared to grade/rate/term, so they're best used to fine-tune pricing within a grade rather than as standalone approval criteria.

*(See [`report/PROJECT_REPORT.md`](report/PROJECT_REPORT.md) for the full write-up with supporting analysis.)*

## 📊 Dashboard Preview

![Dashboard Screenshot](assets/dashboard_screenshot.png)

*Interactive Power BI dashboard — filter by year, loan grade, and term to explore default rate drivers across purpose, interest rate, income, and more.*

## 🗂️ Repository Structure

```
lending-club-default-analysis/
├── README.md
├── LICENSE
├── data/
│   └── loan_csv.csv                  # Raw Lending Club dataset
├── excel/
│   └── LC_Analysis.xlsx              # Pivot tables & slicers
├── notebook/
│   └── Lending_Club_Default_Analysis.ipynb   # Full Python EDA
├── powerbi/
│   └── Lending_club_dashboard.pbix   # Interactive dashboard
├── report/
│   └── PROJECT_REPORT.md             # Full findings & recommendations
└── assets/
    └── dashboard_screenshot.png      # Dashboard preview image
```

## 🛠️ Tools Used

- **Python** (pandas, numpy, matplotlib, seaborn) — data cleaning & exploratory data analysis
- **Microsoft Excel** — pivot tables and slicers for a quick cross-tab view
- **Power BI** — interactive dashboard with cross-filtering by year, grade, and term
- **Jupyter Notebook** — full analysis narrative and code

## 📁 Dataset

The dataset covers loans issued between 2007–2011. Only loans with a final status of `Fully Paid` or `Charged Off` are included (loans still `Current` at time of data extraction were excluded, since their outcome is unknown). Columns unavailable at the time of loan approval (e.g. repayment history, recoveries, last payment date) were removed to avoid data leakage.

## ▶️ How to Reproduce

1. Clone this repo.
2. Open `notebook/Lending_Club_Default_Analysis.ipynb` in Jupyter — all cleaning and analysis steps are documented inline.
3. Open `powerbi/Lending_club_dashboard.pbix` in Power BI Desktop to explore the interactive dashboard.
4. Open `excel/LC_Analysis.xlsx` for the pivot-table view.

## Author

**Vasu Vachhani**
GitHub: [@vvachhani28-ux](https://github.com/vvachhani28-ux)

## License

This project is licensed under the [MIT License](LICENSE).
