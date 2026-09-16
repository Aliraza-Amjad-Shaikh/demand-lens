# DemandLens — Stage 2 EDA Findings

Dataset: Rossmann Store Sales (`train.csv` merged with `store.csv` on `Store`, left join). Final row count: 1,017,209. Verified date range: 2013-01-01 to 2015-07-31. 2015 is a partial year (data ends July 31).

All sales-demand comparisons use `open_days` (`Open == 1`) unless the question specifically concerns availability/closure, in which case the full `train` dataframe is used.

---

## Promo effect on sales

**Question:** Do promotions increase sales, and is the effect consistent across stores and store formats?

**Method:** Compared open-day sales on promo vs. non-promo days, pooled and within-store (all 1,115 stores had both promo and non-promo open days), then broken out by StoreType.

**Result:** Pooled: promo days average 8,228 vs. 5,929 non-promo (open days only) — ~38.8% raw lift. Within-store: median lift 40.6%, IQR 29.4%–51.8%. By StoreType: a strongest (median lift 43.96%), d and c substantial (36.5%, 34.0%), b lower and highly variable (median 10.25%, only 17 stores).

**Business interpretation:** Promotions produce a large, robust sales lift that holds at the individual-store level, not just in aggregate — ruling out "larger stores just run more promos" as the explanation.

**Stage 3/4 implication:** Promo is a strong, reliable feature candidate. StoreType-specific promo-lift terms (interaction features) are justified given the format-level variation, except for StoreType b.

**Caveat:** Assortment b exists almost entirely within StoreType b — StoreType and Assortment are not separable for that segment. Do not claim "StoreType b doesn't respond to promos" as a clean causal statement; the sample (17 stores) is too small and confounded to support it.

---

## CompetitionDistance and sales

**Question:** Does proximity to competitors relate to store sales?

**Method:** Aggregated CompetitionDistance to one row per store (a static attribute) before correlating against average store sales. Applied log1p transform given strong right skew (mean 5,405 vs. median 2,325). Checked StoreType as a confound.

**Result:** Raw distance: no detectable correlation with average sales. Log distance: small, statistically detectable negative correlation (r = -0.1046, p < 0.001), explaining roughly 1% of variance. Within-StoreType: only StoreType a shows a reliable negative association (r = -0.1335, p = 0.001); no reliable relationship for c or d; b too small to judge. StoreType b stores sit much closer to competitors on average than StoreType d.

**Business interpretation:** CompetitionDistance is not a consistent cross-format sales driver. The weak overall effect is mainly a StoreType-a phenomenon and should not be read as a general "competition hurts sales" rule.

**Stage 3/4 implication:** If used as a feature, CompetitionDistance should likely be modeled with a StoreType interaction rather than as a universal linear effect. Not a priority feature on its own.

**Caveat:** Correlation, not causation — proximity may reflect location/format choices made for other reasons (e.g., urban density) rather than competition directly suppressing demand. The 3 stores with null CompetitionDistance (291, 622, 879) were profiled against StoreType×Assortment peers: 291 and 879 share StoreType d/Assortment a but sit on opposite sides of their segment's sales distribution; 622 is a/c and below-segment — they are not a homogeneous group, so a segment-median imputation (deferred to Stage 3/4) can only approximate, not recover, the true values.

---

## StateHoliday and store availability/sales

**Question:** How do state holidays affect whether stores open and how much they sell when open?

**Method:** Compared open rates by StateHoliday type on the full dataset, then open-day sales by StateHoliday and by StoreType × StateHoliday, checking cell sizes throughout.

**Result:** Open rate collapses on holidays: 85.53% ordinary days → 3.43% public holiday → 2.17% Easter → 1.73% Christmas. Conditional on being open, holiday sales are higher on average, but holiday open-day samples are small (694 / 145 / 71 for public holiday / Easter / Christmas respectively). By StoreType, b stays high-selling across all holiday types when open; a and d often dip below their own ordinary-day baseline. Several StoreType×StateHoliday holiday cells are extremely sparse (e.g., Christmas type a = 4 rows, type d = 1 row).

**Business interpretation:** State holidays are overwhelmingly closure days. The higher average sales seen on the few days stores do open is most plausibly explained by selective opening — a non-random, likely higher-demand subset of stores choosing to trade — not a universal holiday-demand effect. This is an unconfirmed hypothesis, not a dataset-verified fact.

**Stage 3/4 implication:** StateHoliday should primarily be modeled as an availability/closure feature. Any holiday-sales-lift feature must not be generalized from the sparse StoreType×StateHoliday cells.

**Caveat:** Never generalize from cells with single-digit counts (e.g., Christmas type d = 1 row). The "selective opening" explanation is a leading hypothesis, not confirmed.

---

## SchoolHoliday and open-day sales

**Question:** Are stores open and do they sell differently during school holidays, after separating availability from customer demand?

**Method:** Compared row counts and open rates by SchoolHoliday on the full dataset, then Sales on open days only, reviewed the distribution with a boxplot, checked results by StoreType, and repeated the comparison after excluding StateHoliday days.

**Result:** Stores were open more often on school-holiday days (91.39%) than on other days (81.34%) — the opposite direction from StateHoliday. Among open days, mean Sales were 7,371.67 during school holidays versus 7,081.43 otherwise (~4.1% lift); median Sales were 6,745 vs. 6,513. Standard deviations were nearly identical (boxplot shapes matched), indicating a shift in center, not spread. The positive lift appeared for StoreTypes a (+5.24%), c (+2.66%), and d (+3.48%); StoreType b showed a small decline (-1.24%) but represents only 17 stores. After excluding StateHoliday days, the mean-sales difference was virtually unchanged (7,368.45 vs. 7,079.53).

**Business interpretation:** School holidays are not primarily a closure signal. Open-day sales are modestly higher during school holidays, and this is not explained by overlap with StateHoliday. The effect is an order of magnitude smaller than the Promo effect.

**Stage 3/4 implication:** SchoolHoliday is a usable but low-magnitude feature candidate relative to Promo. Effect appears broadly consistent across the well-sampled StoreTypes (a, c, d).

**Caveat:** Descriptive, not causal — school-holiday timing may coincide with seasonal demand, weekday composition, or promo activity not yet fully isolated. StoreType b's result (17 stores) should not be generalized.

---

## Day-of-week patterns

**Question:** Do stores trade and sell differently across days of the week, and how does promo scheduling relate to this?

**Method:** Compared open rates by DayOfWeek on the full dataset (availability), then Sales on open days by DayOfWeek (demand), then promo rate by DayOfWeek as a confound check.

**Result:** Stores are open on 92%+ of Monday–Saturday records but only 2.58% of Sunday records (1,418 open observations out of 54,955). Among open days, Monday shows the highest mean sales among well-sampled days (8,403.10) and Saturday the lowest (6,025.07); Sunday's mean (8,778.51) is nominally higher but drawn from the small, high-variance sample and is not generalizable. Promo rate is 54–58% Monday–Friday and exactly 0% on Saturday and Sunday. Wednesday and Thursday have similar promo rates (55.74% vs. 58.20%) but different sales (6,847.96 vs. 6,986.25), showing promo rate alone does not fully explain weekday sales ordering.

**Business interpretation:** Day-of-week and promo scheduling are structurally entangled — promo never runs on weekends — but day-of-week carries sales signal independent of promo, as shown by the Wednesday/Thursday divergence.

**Stage 3/4 implication:** Day-of-week and Promo should be modeled as separate features, potentially with interaction terms. A same-weekday promo-split comparison would further isolate day-of-week's pure effect.

**Caveat:** Sunday's sales figures are based on a small, high-variance sample and are not reliable for generalization. No causal decomposition of day-of-week vs. promo has been performed.

---

## Monthly/yearly seasonality and promo-timing overlap

**Question:** Is there a recurring monthly/yearly sales pattern, and does it align with promo scheduling?

**Method:** Parsed Date to Year/Month on the full, verified dataset (Jan 2013–Jul 2015), computed mean Sales per open store-day by Year-Month (chosen over total sales to avoid conflating store-count changes with demand), plotted the series, and compared against monthly promo rate across all available years.

**Result:** December is the clear seasonal peak in both years fully observed: December 2013 (8,613.46) and December 2014 (8,603.81) are the two highest points in the 31-month series and nearly identical to each other. January is the corresponding low point both years (6,239.64 in 2013; 6,539.40 in 2014). Promo rate in both Decembers was mid-range (41.45% in 2013, 39.80% in 2014) — not elevated, so the December peak is not explained by heightened promo activity. November 2014 shows an anomalous promo-rate spike (61.01%, versus 38.98% in November 2013) alongside a steep sales rise into December, while November 2013's rise is milder without that promo spike.

**Business interpretation:** The December sales peak is a repeating, calendar-linked seasonal effect — most plausibly holiday-shopping-related — confirmed across two independent years, and occurs despite average promo activity, not because of it. November's apparent seasonal rise is more ambiguous, since its 2014 instance is entangled with a one-off promo spike not present in 2013.

**Stage 3/4 implication:** A December/year-end seasonal feature is a strong, evidence-backed candidate independent of promo. November's seasonal strength should not be assumed equal to December's until promo and month effects can be separated (e.g., a same-month promo-split comparison).

**Caveat:** 2015 has no December (data ends July 31, 2015) — the pattern is confirmed across two years, not three. The dataset has no explicit field indicating retailer intent (e.g., marketing calendars), so "why" December peaks remains a plausible interpretation, not a proven cause.

---

## Promo timing vs. weekly/monthly seasonality — confound check

**Question:** Do apparent day-of-week and monthly sales patterns simply reflect when promotions are scheduled, or is there independent signal?

**Method:** Compared promo rate against mean open-day sales across DayOfWeek and across Year-Month, looking for cases where the two align versus diverge.

**Result:** Promo rate and sales move together in some cases (Monday: highest weekday promo rate and highest weekday sales; Saturday: zero promo and lowest weekday sales) but diverge in others (Wednesday/Thursday have similar promo rates but different sales; December sales peak alongside average, not elevated, promo rate in both observed years).

**Business interpretation:** Promo scheduling is a real but partial explanation for weekly and monthly sales variation. Day-of-week and calendar-month carry demand signal independent of promo timing.

**Stage 3/4 implication:** Day-of-week, month, and Promo should be modeled as separate features, potentially with interaction terms, rather than treating one as a proxy for another.

**Caveat:** This is an overlap/divergence check, not a causal decomposition — no regression or variance-partitioning has been performed to quantify independent contributions.

---

## StoreType and Assortment profiles

**Question:** Do store formats and assortment levels differ in sales, customer traffic, and spend-per-customer, and how confounded are StoreType and Assortment?

**Method:** On open days, computed mean/median Sales and Customers by StoreType, derived a diagnostic Sales/Customer ratio (exploratory only, not an engineered feature), and cross-tabulated StoreType × Assortment row counts before interpreting Assortment separately.

**Result:** StoreType b shows far higher mean sales (10,707.79) and mean customers (2,068.49) than a, c, or d, but the lowest sales-per-customer (5.18) — its high sales come from traffic volume, not per-visit spend. StoreType d shows the opposite: lower customer counts (617.05) but the highest sales-per-customer (11.48). Types a and c are similar to each other on this ratio (8.88 and 8.68). The cross-tab confirms Assortment b appears exclusively within StoreType b (3,216 of that row's rows; 0 elsewhere) — Assortment and StoreType are not independent, most severely for segment b, which itself splits across Assortment a (2,631 rows), b (3,216), and c (376).

**Business interpretation:** Store format drives genuinely different customer-behavior profiles — some formats are traffic-driven, others basket-value-driven. This is an operational difference, not merely a scale difference.

**Stage 3/4 implication:** StoreType is a strong categorical feature candidate. Sales/Customer should not be engineered forward as a predictive feature (Customers is not known at prediction time), but the traffic-vs-basket-value insight can inform feature grouping. Assortment b and StoreType b should be treated as one combined segment, never modeled as independent effects.

**Caveat:** StoreType b represents only 17 stores and 6,223 open-day rows — the smallest, most variable segment. Any StoreType-b or Assortment-b finding describes this specific small group, not a generalizable format-level pattern.

---

## Closure-gap characterization

**Question:** Is there a long, shared contiguous closure pattern across many stores, and what does it look like in detail?

**Method:** Built per-store contiguous closure runs from Open==0 rows sorted by date; examined the full distribution of run lengths; profiled the longest-running stores by StoreType/Assortment/CompetitionDistance; checked sales immediately before and after the longest closures.

**Result:** A previously documented "180 stores, 184-day gap" does not match this dataset. Only 2 stores (103 and 1081) show a long, near-identical contiguous closure, both spanning 2013-01-01 to 2013-07-04 (185 days). A third store (349) shares the same start date but closes for only 101 days (ending 2013-04-11) — possibly a coincidental shared start-of-year effect rather than the same event; no other stores show a comparable long run. The two 185-day stores differ in StoreType (d vs. b), Assortment (c vs. a), and CompetitionDistance (5,210 vs. 400), so the pattern is not tied to store format. Both closures fall at the very start of the dataset's date range, so there is no pre-closure sales baseline (n=0 for "before" in both cases). Post-reopening mean sales were 4,575.61 (store 103, n=77) and 5,217.97 (store 1081, n=91).

**Business interpretation:** This looks like a data-coverage artifact — these two stores likely were not yet trading, or not yet included in reporting, when the observation window began, rather than a mid-life renovation event. Open alone cannot distinguish "new store not yet open" from "existing store closed for renovation."

**Stage 3/4 implication:** Per-store lag/rolling feature construction must use each store's own first available Open==1 date as its effective start point, not assume a common 2013-01-01 start for all stores. No universal 184-day/180-store gap adjustment is needed, since that pattern is not present in this dataset as originally documented.

**Caveat:** This finding contradicts a previously documented Stage 1 ground-truth note; that note should be treated as unverified/incorrect for this dataset version. Only 2–3 stores show this left-censored pattern; no causal claim is made about why these specific stores start late.

---

## Dataset-scope note

During Phase 2, a transient error caused the monthly-seasonality analysis to run against a truncated view of the data (Jul 2014–Jul 2015 instead of the full Jan 2013–Jul 2015 range). This was caught, the underlying dataframe was re-verified (full row count 1,017,209, correct date range confirmed), and the monthly-seasonality and promo-overlap findings above were recomputed on the corrected data. Earlier intermediate figures based on the truncated window are superseded by the findings in this document.