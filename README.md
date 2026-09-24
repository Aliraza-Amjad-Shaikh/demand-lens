# DemandLens — Retail Demand Forecasting + AI Business Advisor

DemandLens is an end-to-end retail demand forecasting system built on the Rossmann Store Sales dataset. The goal is not just to predict store sales, but to explain those predictions in plain English — turning raw model output into something a store manager or business analyst can actually use.

This is being built fully in public, stage by stage, with every decision documented. The repo will grow from a cleaned dataset (Stage 1) through a full exploratory analysis (Stage 2) into a forecasting pipeline with an AI narrative layer and a shipped dashboard product (Stage 7).

## Why This Project Exists

Most forecasting projects stop at a notebook with an R² score and a shrug. DemandLens is designed to be closer to what a real retail data team would ship:

- A clean, documented data pipeline that treats data quality as engineering, not a formality
- Exploratory analysis that tests real business hypotheses instead of producing a chart dump
- Models that are evaluated not just on accuracy, but on where they break and why
- An AI layer that narrates forecasts without hallucinating — grounded in actual numbers, not vague storytelling
- A dashboard that a stranger can open and understand without you explaining it

## Dataset

Rossmann Store Sales (Kaggle competition):

- `train.csv`: 1,017,209 daily sales records across 1,115 stores (2013-01-01 to 2015-07-31, with 2015 a partial year ending July 31)
- `test.csv`: 41,088 rows for the held-out forecast window
- `store.csv`: Store-level metadata (StoreType, Assortment, CompetitionDistance, Promo2 participation, etc.)

Competition page: https://www.kaggle.com/c/rossmann-store-sales

## Project Structure

```
demandlens/
├── data/
│   ├── raw/              # Original Kaggle files (gitignored)
│   └── processed/        # Cleaned, merged datasets (gitignored)
├── notebooks/
│   ├── 01_data_cleaning_eda.ipynb
│   └── 02_eda.ipynb
├── docs/
│   ├── decision_log.md   # Running record of every cleaning/EDA decision
│   └── eda_findings.md   # Stage 2 findings: question, method, result, interpretation
├── .gitignore
└── README.md
```

Raw and processed CSVs are gitignored — the pipeline is reproducible from the original Kaggle files, and the repo stays lean.

## Stage Roadmap

This project is being built in 7 deliberate stages. Each stage has a clear "done" criterion and ships something tangible.

### Stage 1 — Data Collection & Cleaning ✅ **COMPLETE**

*Goal:* Turn raw, messy retail data into something trustworthy.

*What was done:*

- Merged `train.csv` + `store.csv` on `Store` (left join), validated row counts and relational integrity
- Fixed `StateHoliday` mixed-type issue (integer `0` + string `'0'` unified to string)
- Imputed 11 missing `Open` values in `test.csv` (Store 622, Sept 2015) as `Open = 1`, documented as assumption
- Confirmed `Promo2Since*`/`PromoInterval` nulls are structural (all where `Promo2 == 0`) — left as `NaN`
- Left `CompetitionDistance` nulls (3 stores: 291, 622, 879) untouched — deferred imputation strategy to Stage 3/4 modeling decision
- Added `competition_open_date_known` flag for stores with unknown competition opening date
- Investigated and documented: 54 zero-sales-while-open rows (consistent with zero customers), and an initial note on a "180-store, 184-day gap" closure pattern (later corrected in Stage 2 — see below)
- Saved cleaned, merged datasets to `data/processed/`

*Outputs:*

- `data/processed/train_cleaned.csv` (1,017,209 rows)
- `data/processed/test_cleaned.csv` (41,088 rows)
- `notebooks/01_data_cleaning_eda.ipynb`
- `docs/decision_log.md` (decision-by-decision trail)

*Stage 1 completion criteria met:*

- One clean merged dataset
- Zero unexplained nulls
- Written log of every cleaning decision and why

---

### Stage 2 — Exploratory Data Analysis (EDA) 🚧 **IN PROGRESS**

*Goal:* Understand the business before you model it — every chart is tied to a real question, a method, a defensible result, and a business interpretation, not produced for its own sake.

*Approach:* Hypothesis-driven, not chart-dumping. Stage 1 decisions were treated as locked ground truth going in; where EDA has found genuine evidence contradicting a Stage 1 note, that's been flagged explicitly and corrected in `decision_log.md` rather than silently overwritten (see the closure-gap correction below). No imputation of the deferred `CompetitionDistance` nulls and no outlier removal happens in this stage — EDA gathers evidence for those decisions, which are made in Stage 3/4 once a model family is chosen.

*Findings completed so far* (full detail in `docs/eda_findings.md`):

- **Promo effect on sales** — Promotions produce a large, robust sales lift (~38.8% pooled, ~40.6% median per-store), confirmed at the individual-store level across all 1,115 stores, not just in aggregate. Effect varies by StoreType (strongest for type a); StoreType b's low apparent lift is confounded with Assortment b and can't be read as a clean format effect (only 17 stores, and Assortment b exists almost exclusively within StoreType b).
- **CompetitionDistance and sales** — No relationship on raw distance; a small, statistically detectable negative relationship emerges only after log-transforming (r = -0.10, explaining ~1% of variance), and even that is mainly a StoreType-a phenomenon, not a general rule. The 3 null-distance stores (291, 622, 879) were profiled against StoreType×Assortment peers and found to be a heterogeneous group, not a single pattern.
- **StateHoliday: availability and sales** — State holidays are overwhelmingly closure days (open rate collapses from 85.5% to 1.7–3.4%). Stores that do stay open show higher average sales, most plausibly due to selective opening by a non-random, higher-demand subset — not a universal holiday-demand effect. Many StoreType×StateHoliday cells are too sparse to generalize from.
- **SchoolHoliday and sales** — Unlike StateHoliday, stores are open *more* often during school holidays (91.4% vs. 81.3%), and open-day sales are modestly higher (~4.1% lift), holding up even after excluding StateHoliday overlap. Effect is an order of magnitude smaller than Promo's.
- **Day-of-week patterns** — Stores are open 92%+ of Monday–Saturday but only 2.6% of Sundays. Promo never runs on weekends, entangling day-of-week with promo scheduling — but the Wednesday/Thursday divergence (similar promo rates, different sales) shows day-of-week carries independent signal.
- **Monthly/yearly seasonality** — December is a confirmed, repeating seasonal peak across both fully-observed years (2013, 2014), occurring despite average — not elevated — promo activity that month, ruling out promo as the explanation. January is the corresponding low point both years.
- **Promo timing vs. seasonality confound check** — Day-of-week, month, and Promo each carry independent signal; none is fully explained by the others.
- **StoreType and Assortment profiles** — StoreType b drives sales through customer volume (highest traffic, lowest sales-per-customer); StoreType d drives sales through higher spend-per-visit (lowest traffic, highest sales-per-customer). Assortment b exists almost exclusively within StoreType b — the two variables are not independent for that segment.
- **Closure-gap characterization, with a correction to Stage 1** — The originally documented "180 stores, 184-day gap" does not hold up under direct investigation. Only 2 stores (103, 1081) show a long contiguous closure, both left-censored at the dataset's very start (2013-01-01 to 2013-07-04, 185 days) — consistent with these stores not yet being open/reporting when data collection began, not a mid-life renovation event. This has been logged in `docs/decision_log.md` as an explicit correction, since it directly affects how Stage 3/4 lag/rolling features must be built (per-store effective start date, not a shared closure-window adjustment).

*Remaining work:* Final consolidated review of all findings for internal consistency, and confirming the notebook (`02_eda.ipynb`) runs cleanly top-to-bottom with all findings narrated.

*Done when:* Every finding has a stated question, method, result, and business interpretation; small-sample and confounded results are explicitly flagged rather than generalized; and both deferred Stage 1 items (CompetitionDistance nulls, closure-gap pattern) are fully characterized with evidence — without being resolved — ready to inform Stage 3/4.

---

### Stage 3 — Baseline Machine Learning

*Goal:* Prove a simple model works before reaching for something complex.

*Planned work:*

- Feature engineering: lag features, rolling averages, date encodings, promo flags, competition-distance buckets
- Resolve the two items deferred from Stage 1/2: choose a `CompetitionDistance` imputation strategy for stores 291/622/879 (segment-level median plus an explicit missingness indicator is the leading candidate per EDA evidence), and set each store's lag/rolling feature computation to its own effective start date given the closure-gap correction
- Baseline models: linear regression → random forest → XGBoost
- Naive baseline: "predict tomorrow = same as last week"

*Done when:* Working baseline model, MAE/RMSE reported, clear margin over naive baseline.

---

### Stage 4 — Advanced Forecasting Model

*Goal:* Build the model that's actually good enough to ship.

*Planned work:*

- Time-series methods: Prophet (seasonality/holidays native) or LSTM (if demonstrating deep learning)
- Per-store or per-store-cluster modeling (tradeoff explained) — EDA suggests StoreType/Assortment segments behave differently enough (e.g., traffic-driven vs. basket-value-driven formats) to be a real candidate for segment-aware modeling
- Rolling-window cross-validation (multiple windows, not a single split)

*Done when:* Stable error across ≥3 validation windows, predicted-vs-actual chart tracking well.

---

### Stage 5 — Model Evaluation & Explainability

*Goal:* Prove you understand your model, not just that it runs.

*Planned work:*

- SHAP values: global + local feature importance
- Error analysis: which stores/periods does the model get wrong, and why
- Honest evaluation report: where to trust the model, where not

*Done when:* Written evaluation doc with SHAP visuals and an explicit "here's where this model breaks" section.

---

### Stage 6 — GenAI Layer

*Goal:* Turn model output into something a human actually wants to read.

*Planned work:*

- Local LLM (via Ollama) takes forecast + SHAP + context, generates plain-English weekly brief
- Strict grounding: LLM only narrates numbers fed to it, never invents figures
- Explicit hallucination test: 10 forecasts, verify every number/claim matches source data

*Done when:* 10/10 test briefs fully grounded and factually consistent.

---

### Stage 7 — Product / Software

*Goal:* Make it something a stranger can actually use.

*Planned work:*

- Web dashboard (Streamlit or FastAPI + React): searchable store selector, forecast chart, SHAP breakdown, AI brief, PDF export
- Curated "featured stores" default set
- Clean README, live demo link if deployed

*Done when:* Someone with zero context can open the link, pick a store, and understand what's forecasted and why — without explanation.

---

## How to Reproduce Stage 1 & 2

1. Download `train.csv`, `test.csv`, `store.csv` from the Kaggle competition page into `data/raw/`
2. Run `notebooks/01_data_cleaning_eda.ipynb` from top to bottom to produce the cleaned, merged datasets
3. Run `notebooks/02_eda.ipynb` from top to bottom to reproduce the Stage 2 findings
4. Cleaned outputs will be saved to `data/processed/`

Raw data files are not committed — the notebooks are the reproducible pipeline.

## Decision Log & Findings

All meaningful cleaning/EDA decisions are recorded in `docs/decision_log.md`, including:

- What was changed or decided
- Options considered
- Reasoning and chosen approach
- Downstream impact (if any)
- Explicit labeling of assumptions vs. facts
- Corrections, where later evidence overturned an earlier note (see the Stage 2 closure-gap correction)

Stage 2's full set of findings — question, method, result, business interpretation, and caveats for every investigation — is recorded separately in `docs/eda_findings.md`, written to be usable as direct input to Stage 3/4 feature decisions, not just as internal notes.

Both documents are intended to be LinkedIn/GitHub writeup-ready.

## Tech Stack

- Python, pandas, numpy
- matplotlib for EDA visualization
- scipy for statistical testing (correlation, significance)
- Jupyter/Colab for notebooks
- Git + GitHub for version control

Later stages will add: scikit-learn, XGBoost/LightGBM, Prophet/LSTM, SHAP, Ollama (local LLM), Streamlit/FastAPI.

## License

This is a portfolio/learning project. Rossmann dataset is provided under Kaggle's competition terms.

---

*Built in public by Aliraza*
