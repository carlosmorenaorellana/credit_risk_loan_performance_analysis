# Bank Loan Credit Risk Analysis


## Executive Summary
This project delivers an end-to-end credit risk and loan performance analytics solution using **BigQuery**, **Google Sheets**, and **Looker Studio**. The goal is to evaluate portfolio risk exposure, model expected losses (EL), and provide financial risk committees with interactive macro stress-testing capabilities.

---

## Tech Stack
* **Cloud Data Warehouse:** Google BigQuery (SQL)
* **Financial Modeling & ETL:** Google Sheets / Python
* **Data Visualization & BI:** Looker Studio
* **Version Control:** Git & GitHub

---

## Project Structure

```text
credit_risk_loan_performance_analysis
├── data                    # Raw and processed CSV datasets
│   ├── raw                 # Original credit risk dataset
│   └── processed           # Cleaned and enriched data files
├── outputs                 # Exported charts, summary reports, or presentation assets
├── sheets                  # Workspace exports or financial modeling templates
├── sql_queries             # SQL scripts for data extraction and transformation
├── .gitignore              # Files and directories ignored by Git
├── README.md               # Comprehensive project documentation
└── requirements.txt        # Python dependencies
```

---

## 1. Data Quality & Missing Value Analysis (`loan_int_rate`)

* **The Issue:** The `loan_int_rate` column contained `3,047` missing values (~9.4% of the dataset).
* **Investigation:** Multi-dimensional grouping against `loan_grade`, `loan_intent`, and default indicators revealed a uniform distribution (missing rates hovering between 8.5% and 11% across grades/intents). 
* **Finding:** The data is **Missing Completely at Random (MCAR)** and does not indicate hidden risk bias.
* **Decision:** Preserved records with `NULL` values intact to prevent portfolio distortion and severe data loss.

---

## 2. Data Enrichment & Feature Engineering (`06_create_enriched_table.sql`)

An enriched analytical table (`credit_risk_enriched`) was created in BigQuery to standardize financial metrics:

* **`is_default`:** Binary flag (`1`/`0`) derived from `loan_status` to enable dynamic aggregation and default rate calculation in downstream BI tools.
* **`income_bracket`:** Segmented into Low (<30k), Medium (30k-70k), and High (>70k).
* **`burden_category`:** Risk benchmarks based on `loan_percent_income`:
  * **Healthy:** `< 15%` (~50% of portfolio)
  * **Caution:** `15% - 30%` (~40% of portfolio)
  * **Dangerous:** `> 30%` (~11% of portfolio)

---

## 3. Exploratory Data Analysis & Default Rate Insights

Using Connected Sheets in Google Sheets, dynamic average aggregation on the `is_default` flag yielded the portfolio default rates by **Loan Grade**:

| Loan Grade | Portfolio Default Rate | Risk Interpretation |
| :--- | :--- | :--- |
| **A** | 9.6% | Prime / Low Risk Baseline |
| **B** | 15.9% | Low-Moderate Risk |
| **C** | 20.3% | Moderate Risk |
| **D** | 58.8% | High Risk / Subprime |
| **E** | 64.2% | Very High Risk |
| **F** | 70.3% | Severe Risk |
| **G** | 98.4% | Near-Certain Default |

* **Business Validation:** Empirical results confirm the underwriting model functions as intended—default rates increase sharply from Grade A to Grade G, validating risk grading accuracy.

---

## 4. Risk-Based Pricing Analysis (`loan_int_rate` vs. `loan_grade`)

* **Objective:** Evaluate how lending rates scale across different credit risk tiers.
* **Findings:** Average interest rates increase progressively from **7.3%** in Grade A to **20.3%** in Grade G.
* **Business Validation:** Confirms that the portfolio implements appropriate risk-based pricing, charging higher interest to offset increased default risk in lower credit tiers.

---

## 5. Portfolio Concentration Analysis (`loan_intent` & `person_emp_length`)

* **Capital Allocation by Intent:** Portfolio exposure is evenly diversified across major categories, led by **Education (19.56%)**, **Medical (18.02%)**, and **Venture (17.52%)**, minimizing single-sector concentration risk.
* **Workforce Stability (`person_emp_length`):** Capital volume concentrates heavily among borrowers with 0 to 6 years of employment tenure, forming a standard retail lending risk curve.

---

## 6. Portfolio Volume, Risk Concentration & Strategic Pricing vs. Industry Benchmarks
* **Portfolio-Wide Risk Baseline:** The overall portfolio default rate stands at **21.5%**, which significantly exceeds the standard industry benchmark for high risk (>10%). 
* **Comparison with Market Standards:** 
  * Even the safest segments of the portfolio perform above standard high-risk thresholds. Grade A loans record a **9.6%** default rate (near the upper limit of high risk), while high-income borrowers (>70k) show a minimum average default rate of **17%** (70% higher than the standard high-risk floor).
  * Similarly, the healthiest debt burden category (<15% LTI) registers a **12%** default rate, sitting 20% above the standard high-risk threshold.
* **Volume Distribution & Core Exposure:** 
  * The bulk of the portfolio's volume and transactions moves through Grades A, B, and C, which experience default rates ranging from **5.5% to 27%**. 
  * 95% of total capital originates from medium and high-income brackets, while 48% sits in the caution LTI tier (15%–30%) and 19.1% in the dangerous tier (>30%).
* **The Pricing Disconnect:** Average interest rates remain flat (~11%–12%) across income brackets and risk tiers, failing to scale with true portfolio risk. For instance, the dangerous burden category experiences a 70.6% default rate but receives the same baseline pricing as healthy profiles.
* **Strategic Recommendation:** Because the entire portfolio operates at an elevated baseline compared to retail banking benchmarks, implementing dynamic risk-based pricing is mandatory to adjust interest rates for high-burden and subprime segments, thereby protecting financial margins.

---

## 7. Demographic Age Distribution & Pricing Parity
* **Age Concentration:** Borrower volume and capital exposure are heavily concentrated among younger adults, with **88.9%** of total loans originating from customers aged 18 to 36 ([18–27] at 52.9% and [27–36] at 36.0%).
* **Pricing Parity Across Age Brackets:** Average interest rates remain uniform (~11.0% to 11.2%) across all major working-age brackets up to age 81, showing no risk-based differentiation based on age.
* **Strategic Observation:** While younger cohorts drive primary portfolio liquidity, age does not currently reflect risk-adjusted pricing variance, aligning with the broader static interest rate trend observed across income and burden tiers.

---

## 8. Demographic Age Distribution & Pricing Parity
* **Age Concentration:** Borrower volume and capital exposure are heavily concentrated among younger adults, with **88.9%** of total loans originating from customers aged 18 to 36 ([18–27] at 52.9% and [27–36] at 36.0%).
* **Pricing Parity Across Age Brackets:** Average interest rates remain uniform (~11.0% to 11.2%) across all major working-age brackets up to age 81, showing no risk-based differentiation based on age.
* **Strategic Observation:** While younger cohorts drive primary portfolio liquidity, age does not currently reflect risk-adjusted pricing variance, aligning with the broader static interest rate trend observed across income and burden tiers.

---

## 9. Financial Risk Modeling & Macro Stress Testing
* **Expected Loss (EL) Framework:** Calculated total portfolio risk exposure at **29.67M €** (dynamic BI view) using standard parameters EL = EAD * PD * LGD with a 45% LGD baseline). Note: A static pre-aggregation model built in Google Sheets previously estimated baseline exposure at **31.93M €** due to grouped loan-grade level estimations; the live BigQuery and Looker Studio architecture utilizes row-level evaluation for higher operational precision.
* **Stress Testing & Sensitivity Analysis:** Simulated a macroeconomic shock (+10% default uplift across tiers), demonstrating an additional capital loss impact under stress testing, projecting up to **59.34M €** under severe risk uplifts.
* **Business Value:** Translates statistical default probabilities into monetary risk exposure, providing risk committees with quantitative inputs for capital provisioning.

---

## 10. Data Studio BI Architecture & Visualization Strategy
* **Single Source of Truth (SSOT):** Connected Looker Studio directly to the `credit_risk_enriched` BigQuery table to ensure real-time updates and eliminate redundant offline files.
* **Multi-Page Executive Dashboard:** Designed a professional 3-page reporting suite structured for institutional stakeholders:
  * **Page 1 (Executive Summary):** Global KPIs (Total Volume, Default Rate, Expected Loss) paired with risk distribution by loan grade and intent.
  * **Page 2 (Risk Segmentation):** Heatmap analysis crossing income brackets with debt burden categories, alongside workforce stability metrics.
  * **Page 3 (Macro Stress Test):** Interactive parameter slider allowing risk teams to stress-test portfolio sensitivity and visualize monetary impact instantly.
* **Business Impact:** Empowers executive management with self-service exploratory views, streamlining credit risk monitoring and regulatory capital assessment.