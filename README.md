\# DemandLens — Retail Demand Forecasting + AI Business Advisor



DemandLens is an end-to-end retail demand forecasting system built on the Rossmann Store Sales dataset. The goal is not just to predict store sales, but to explain those predictions in plain English — turning raw model output into something a store manager or business analyst can actually use.



This is being built fully in public, stage by stage, with every decision documented. The repo will grow from a cleaned dataset (Stage 1) into a full forecasting pipeline with an AI narrative layer and a shipped dashboard product (Stage 7).



\## Why This Project Exists



Most forecasting projects stop at a notebook with an R² score and a shrug. DemandLens is designed to be closer to what a real retail data team would ship:



\- A clean, documented data pipeline that treats data quality as engineering, not a formality

\- Models that are evaluated not just on accuracy, but on where they break and why

\- An AI layer that narrates forecasts without hallucinating — grounded in actual numbers, not vague storytelling

\- A dashboard that a stranger can open and understand without you explaining it



\## Dataset



Rossmann Store Sales (Kaggle competition):

\- `train.csv`: 1,017,209 daily sales records across 1,115 stores (2013-01-01 to 2015-07-31)

\- `test.csv`: 41,088 rows for the held-out forecast window

\- `store.csv`: Store-level metadata (StoreType, Assortment, CompetitionDistance, Promo2 participation, etc.)



Competition page: https://www.kaggle.com/c/rossmann-store-sales



\## Project Structure



```

demandlens/

├── data/

│   ├── raw/              # Original Kaggle files (gitignored)

│   └── processed/        # Cleaned, merged datasets (gitignored)

├── notebooks/

│   └── 01\_data\_cleaning\_eda.ipynb

├── docs/

│   └── decision\_log.md   # Running record of every cleaning/EDA decision

├── .gitignore

└── README.md

```



Raw and processed CSVs are gitignored — the pipeline is reproducible from the original Kaggle files, and the repo stays lean.



\## Stage Roadmap



This project is being built in 7 deliberate stages. Each stage has a clear "done" criterion and ships something tangible.



\### Stage 1 — Data Collection \& Cleaning ✅ \*\*COMPLETE\*\*



\*\*Goal:\*\* Turn raw, messy retail data into something trustworthy.



\*\*What was done:\*\*

\- Merged `train.csv` + `store.csv` on `Store` (left join), validated row counts and relational integrity

\- Fixed `StateHoliday` mixed-type issue (integer `0` + string `'0'` unified to string)

\- Imputed 11 missing `Open` values in `test.csv` (Store 622, Sept 2015) as `Open = 1`, documented as assumption

\- Confirmed `Promo2Since\*`/`PromoInterval` nulls are structural (all where `Promo2 == 0`) — left as `NaN`

\- Left `CompetitionDistance` nulls (3 stores) untouched — deferred imputation strategy to Stage 3/4 modeling decision

\- Added `competition\_open\_date\_known` flag for stores with unknown competition opening date

\- Investigated and documented: 54 zero-sales-while-open rows (consistent with zero customers), 180-store 184-day gap pattern (likely planned closures)

\- Saved cleaned, merged datasets to `data/processed/`



\*\*Outputs:\*\*

\- `data/processed/train\_cleaned.csv` (1,017,209 rows)

\- `data/processed/test\_cleaned.csv` (41,088 rows)

\- `notebooks/01\_data\_cleaning\_eda.ipynb`

\- `docs/decision\_log.md` (decision-by-decision trail)



\*\*Stage 1 completion criteria met:\*\*

\- One clean merged dataset

\- Zero unexplained nulls

\- Written log of every cleaning decision and why



\---



\### Stage 2 — Exploratory Data Analysis (EDA) 🚧 \*\*IN PROGRESS\*\*



\*\*Goal:\*\* Understand the business before you model it.



\*\*Planned work:\*\*

\- Sales trends by day-of-week, month, holiday, StoreType, Assortment

\- Promo vs. non-promo sales comparison (does Promo2 actually work? for which store types?)

\- Competition distance vs. sales correlation

\- Anomaly identification: store closures, extreme outliers, unusual patterns



\*\*Done when:\*\* 8–10 genuine, defensible insights, each backed by a chart and a one-line business interpretation.



\---



\### Stage 3 — Baseline Machine Learning



\*\*Goal:\*\* Prove a simple model works before reaching for something complex.



\*\*Planned work:\*\*

\- Feature engineering: lag features, rolling averages, date encodings, promo flags, competition-distance buckets

\- Baseline models: linear regression → random forest → XGBoost

\- Naive baseline: "predict tomorrow = same as last week"



\*\*Done when:\*\* Working baseline model, MAE/RMSE reported, clear margin over naive baseline.



\---



\### Stage 4 — Advanced Forecasting Model



\*\*Goal:\*\* Build the model that's actually good enough to ship.



\*\*Planned work:\*\*

\- Time-series methods: Prophet (seasonality/holidays native) or LSTM (if demonstrating deep learning)

\- Per-store or per-store-cluster modeling (tradeoff explained)

\- Rolling-window cross-validation (multiple windows, not a single split)



\*\*Done when:\*\* Stable error across ≥3 validation windows, predicted-vs-actual chart tracking well.



\---



\### Stage 5 — Model Evaluation \& Explainability



\*\*Goal:\*\* Prove you understand your model, not just that it runs.



\*\*Planned work:\*\*

\- SHAP values: global + local feature importance

\- Error analysis: which stores/periods does the model get wrong, and why

\- Honest evaluation report: where to trust the model, where not



\*\*Done when:\*\* Written evaluation doc with SHAP visuals and an explicit "here's where this model breaks" section.



\---



\### Stage 6 — GenAI Layer



\*\*Goal:\*\* Turn model output into something a human actually wants to read.



\*\*Planned work:\*\*

\- Local LLM (via Ollama) takes forecast + SHAP + context, generates plain-English weekly brief

\- Strict grounding: LLM only narrates numbers fed to it, never invents figures

\- Explicit hallucination test: 10 forecasts, verify every number/claim matches source data



\*\*Done when:\*\* 10/10 test briefs fully grounded and factually consistent.



\---



\### Stage 7 — Product / Software



\*\*Goal:\*\* Make it something a stranger can actually use.



\*\*Planned work:\*\*

\- Web dashboard (Streamlit or FastAPI + React): searchable store selector, forecast chart, SHAP breakdown, AI brief, PDF export

\- Curated "featured stores" default set

\- Clean README, live demo link if deployed



\*\*Done when:\*\* Someone with zero context can open the link, pick a store, and understand what's forecasted and why — without explanation.



\---



\## How to Reproduce Stage 1



1\. Download `train.csv`, `test.csv`, `store.csv` from the Kaggle competition page into `data/raw/`

2\. Run `notebooks/01\_data\_cleaning\_eda.ipynb` from top to bottom

3\. Cleaned outputs will be saved to `data/processed/`



Raw data files are not committed — the notebook is the reproducible pipeline.



\## Decision Log



All meaningful cleaning/EDA decisions are recorded in `docs/decision\_log.md`, including:

\- What was changed

\- Options considered

\- Reasoning and chosen approach

\- Downstream impact (if any)

\- Explicit labeling of assumptions vs. facts



This is intended to be LinkedIn/GitHub writeup-ready — not just internal notes.



\## Tech Stack



\- Python, pandas, numpy

\- Jupyter/Colab for notebooks

\- Git + GitHub for version control



Later stages will add: scikit-learn, XGBoost/LightGBM, Prophet/LSTM, SHAP, Ollama (local LLM), Streamlit/FastAPI.



\## License



This is a portfolio/learning project. Rossmann dataset is provided under Kaggle's competition terms.



\---



\*\*Built in public by Aliraza\*\*  

```



