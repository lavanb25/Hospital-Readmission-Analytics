# 🏥 Hospital Readmission Analytics

> End-to-end analysis of 30-day hospital readmissions in 100k+ diabetic patient encounters, using **SQL (DuckDB)**, **Python**, **statistics**, and an **interactive Streamlit dashboard**.

**🔗 Live dashboard:** _add your Streamlit Community Cloud link here_

![Odds ratios](images/08_odds_ratios.png)

## 1. Business problem
Hospitals are penalised for avoidable 30-day readmissions, and each one is costly. **Which patients are most at risk, and what can management do about it before discharge?**

## 2. Data
UCI *Diabetes 130-US Hospitals (1999-2008)*: 101,766 encounters, 50 features. See [`data/README.md`](data/README.md) for download steps.

## 3. Approach
| Step | What I did | Tools |
|---|---|---|
| Clean | Handled `?` missing values, removed deceased/hospice discharges (cannot be readmitted), grouped ICD-9 codes into 9 diagnosis categories, built age bands and prior-visit features | Python (pandas) |
| Query | 12 SQL queries: aggregations, CTEs, window functions (`RANK`, `ROW_NUMBER`, `NTILE`, running totals), percentile outlier detection | DuckDB SQL |
| Explore | Readmission rate by age, diagnosis, admission type, HbA1c testing, prior visits, stay length | matplotlib, seaborn |
| Test | Chi-square (categorical), Welch t-test (numeric), effect sizes (Cramér's V, Cohen's d) | scipy |
| Model | Logistic regression on first encounters per patient to estimate **odds ratios** (interpretation over prediction) | statsmodels |
| Communicate | Interactive dashboard with filters + management recommendations | Streamlit, Plotly |

## 4. Key findings
> ✏️ Fill these in with YOUR real numbers after running the pipeline (`outputs/`, `images/`).

- Overall 30-day readmission rate: **__%** across __ encounters
- Patients with 4+ prior inpatient visits: **__%** vs **__%** for none (odds ratio __)
- Highest-risk diagnosis group: **__** (**__%**); biggest contributor to total readmissions: **__** (**__%** of all)
- Medication change vs no change: __% vs __% (chi-square p = __, effect size __)
- Model test AUC **__**: modest, which tells us readmission depends on factors not captured here, so I use the model to *explain*, not to predict

## 5. Recommendations
1. Flag patients with multiple prior inpatient stays for follow-up within 72 hours of discharge.
2. Prioritise discharge planning resources on the diagnosis groups driving most readmissions.
3. Review discharge destinations with the highest odds ratios.
4. Pilot (do not mandate) standardised HbA1c testing, since the finding is an association, not proof of causation.

## 6. Limitations
Observational data (1999-2008), diabetes population only, no cost data, diagnosis simplified from ICD-9 codes. Associations are not causal.

## 7. Project structure
```
├── data/               raw (not committed), processed (committed), README
├── sql/                01_basic_analysis.sql, 02_advanced_analysis.sql
├── src/                01_clean_data.py → 04_stats_and_model.py
├── app/dashboard.py    Streamlit dashboard
├── images/             charts used in this README
├── outputs/            SQL results, test results, odds ratios (CSV)
└── docs/               business summary template, interview prep
```

## 8. How to run
```bash
git clone https://github.com/lavan-crypto/hospital-readmission-analytics.git
cd hospital-readmission-analytics
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# put diabetic_data.csv in data/raw/ (see data/README.md), then:
python src/01_clean_data.py       # clean + load into DuckDB
python src/02_run_sql.py          # run all SQL queries -> outputs/sql_results/
python src/03_eda_charts.py       # charts -> images/
python src/04_stats_and_model.py  # tests + logistic regression
streamlit run app/dashboard.py    # interactive dashboard
```

## 9. Sample SQL
```sql
-- Top 3 specialties per diagnosis group by readmission rate (CTE + window function)
WITH base AS (
    SELECT diag_group, medical_specialty, COUNT(*) AS n, AVG(readmitted_30d) AS rate
    FROM encounters
    WHERE medical_specialty <> 'Unknown'
    GROUP BY diag_group, medical_specialty
    HAVING COUNT(*) >= 100
)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY diag_group ORDER BY rate DESC) AS rn FROM base
) WHERE rn <= 3;
```

## 10. Author
**Lavanyaa B S**: M.Tech AI & Data Science, SASTRA | [LinkedIn](https://linkedin.com/in/lavanyaa-baswa-85366823b) | [GitHub](https://github.com/lavan-crypto)
