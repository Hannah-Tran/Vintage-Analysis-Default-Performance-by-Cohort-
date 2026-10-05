# Vintage Analysis of Consumer Loan Default Using SQL

## 📊 Project Overview
This project analyses over 2 million peer-to-peer loans from the Lending Club dataset (2007–2018) by origination cohort ("vintage"). It examines whether newer cohorts perform worse than older ones at the same age, when in a loan's life default occurs, whether underwriting standards changed as the lender grew, and whether each cohort was priced to cover the losses it produced. The analysis applies survival and hazard-rate concepts from the CQF credit risk module and the PD × LGD × EAD framework, implemented entirely in SQL.

## 🎯 Objective
To replicate the vintage monitoring a retail credit risk team performs: building like-for-like vintage curves, estimating lifetime probability of default from an empirical survival curve, separating underwriting effects from macroeconomic effects, and testing pricing adequacy, presented as a portfolio-ready case study.

## 🧰 Tools Used
- SQL (SQLite via DB Browser for SQLite): CTEs, window functions, recursive CTEs
- GitHub (project documentation)
- Dataset: [Lending Club Loan Data 2007–2018](https://www.kaggle.com/datasets/wordsforthewise/lending-club) via Kaggle

## ⚠️ The Core Problem: Right-Censoring
The data ends in late 2018. A loan issued in 2012 has completed its full life, while a loan issued in 2017 has had only one to two years to default. A simple "default rate by year" therefore makes recent vintages look safer than they really are. Every query in this analysis either compares vintages **at the same age** or explicitly adjusts for censoring.

## 📌 Key Business Questions
1. How mature is each vintage, and how much of it has reached a final outcome?
2. Why is a simple default rate by issue year misleading?
3. Are newer cohorts performing better or worse than older ones at the same age?
4. When in a loan's life does default happen, and what is the lifetime probability of default?
5. Did underwriting standards loosen as origination volume grew?
6. Were weak vintages caused by lending to riskier borrowers, or by the economic environment?
7. How much is lost when a loan defaults (EAD, recovery, LGD)?
8. Was each vintage priced to cover the credit losses it produced?

## 🔍 Analysis Structure

| Query | Business Question | Technique |
|-------|-------------------|-----------|
| 0 | Setup: loan age and months on book | Text-date parsing, `CREATE TABLE` helper table |
| 1 | Vintage maturity | Share of loans paid, defaulted and still open |
| 2 | Why naive default rates mislead | Naive versus resolved-only default rate |
| 3 | Like-for-like cohort comparison | Cumulative default rate at 6, 12, 24, 36 and 48 months, by term |
| 4 | Timing of default and lifetime PD | Life table: marginal PD, hazard rate, survival curve (recursive CTE) |
| 5 | Underwriting drift | FICO, DTI, grade E–G share and 60-month share by vintage |
| 6 | Underwriting versus macro effect | Mix-adjusted performance index (actual vs expected from grade mix) |
| 7 | Loss severity | EAD, recovery rate, LGD and net loss rate by vintage |
| 8 | Pricing adequacy | Credit triangle: break-even spread = annual default × LGD |

## 🧠 Methodology Notes
- **Months on book:** dates stored as text (e.g. `Dec-2015`) are converted to a month index to calculate loan age and the month each loan left observation
- **Vintage curves:** a loan is included in the N-month column only if it is at least N months old, so every vintage is compared at the same age
- **Life-table hazard:** fully paid and still-open loans are treated as censored, with the standard actuarial half-interval adjustment. Survival is the running product of (1 − marginal PD), the discrete form of `P(0,T) = exp(−∫λ dt)`
- **Mix adjustment:** each vintage's actual 12-month default is compared with the rate expected from its grade mix, separating credit-selection effects from environmental effects
- **Loss severity:** EAD = funded amount − principal repaid; recovery is net of collection fees. Recent vintages are still collecting recoveries, so their LGD is overstated
- **Default timing proxy:** the last payment date marks the start of distress; formal charge-off typically follows four to five months later


## 💡Concepts Applied
- **Survival probability and hazard rate:** `P(0,T) = exp(−∫λ dt)` and `λᵢ = −(1/τ)·ln(P(Tᵢ)/P(Tᵢ₋₁))` (CQF Lecture 5.7, Estimating Default Probability), estimated empirically from loan outcomes
- **Lifetime PD:** cumulative default at contract maturity, as used in IFRS 9 Stage 2 provisioning
- **Expected Loss (PD × LGD × EAD):** each component measured by vintage
- **Credit triangle:** `spread ≈ λ × (1 − RR)` (CQF M5S8), used as a break-even pricing check
- **ECL / CVA analogy:** lifetime ECL shares the CVA structure `(1 − R) × ∫ EE × DF × dPD` (CQF M5S5); marginal PDs from the life table supply the dPD term
- **Credit cycle analysis:** separating underwriting quality from macroeconomic conditions

## ✅ How to Reproduce
1. Download the Lending Club dataset from [Kaggle](https://www.kaggle.com/datasets/wordsforthewise/lending-club)
2. Open **DB Browser for SQLite** → New Database → name it `credit_portfolio.db`
3. Import the CSV: **File → Import → Table from CSV file** (tick "Column names in first line"). The table is named `accepted_2007_to_2018Q4` automatically
4. In the **Execute SQL** tab, run **Query 0** once to build the `vintage_base` helper table, then click **Write Changes**
5. Highlight one query at a time and press **F5**

## 💼 Author
**Hannah (Huong) Tran** |
Financial Analysis |
MSc Banking & Finance with Distinction |
CFA Levels I & II cleared

[LinkedIn](https://www.linkedin.com/in/huong-hannah/) | [GitHub](https://github.com/Hannah-Tran)
