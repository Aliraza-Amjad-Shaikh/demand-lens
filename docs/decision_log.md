Decision: Keep the 54 rows where Open == 1 and Sales == 0 (and Customers == 0) unmodified.



Options considered: (A) leave as-is, (B) drop rows, (C) add an anomaly-flag column.



Reasoning: Rows are internally consistent (zero customers → zero sales, not contradictory), affect only \~0.005% of train.csv, and dropping/flagging would add complexity or create time-series gaps with no measurable benefit.



Chosen approach: Option A — no transformation.



Downstream impact: None expected; noted as an accepted rarity, not a resolved data error, since root cause (weather, disruption, etc.) can't be confirmed from data alone.







Decision: Document the 180-store, 184-day gap pattern; take no corrective action in Stage 1.



Options considered: (A) document only, (B) impute/fill gap rows, (C) drop affected stores entirely.



Reasoning: Gaps are contiguous, identical in length across stores, and bookended by clean data — consistent with real store closures (e.g., renovation), not data corruption. Nothing here is factually wrong to correct; imputing would fabricate data that never existed.



Chosen approach: Option A — record store IDs, gap length (184 days), and pattern in documentation.



Downstream impact — flagged, not solved: Lag/rolling-window features (Stage 3–4) must be computed per-store on the actual date index, not by raw row-position, or they will silently pull pre-closure values across the gap. This is a feature-engineering concern, not a Stage 1 fix.



Labeled as assumption, not fact: The specific cause (renovation vs. another systematic closure reason) is inferred from the pattern, not confirmed externally.







Decision: Impute the 11 missing Open values in test.csv (Store 622, Sept 5–17 2015, non-Sunday days) as Open = 1.



Options considered: (A) impute as 1 (assume open), (B) impute as 0 (assume closed), (C) leave null for downstream handling.



Reasoning: Surrounding days for the same store show normal trading; this is a well-precedented resolution for this specific known Rossmann test-set quirk; leaving null risks breaking downstream feature engineering.



Chosen approach: Option A.



Labeled as assumption, not fact: We don't know the actual root cause of the missingness (data collection gap vs. genuine uncertainty about store status) — imputing 1 is a defensible inference, not a confirmed truth.



Downstream impact: None expected beyond this — Open is now fully populated across test.csv, consistent with train.csv.







Decision 1 — Promo2 group nulls (Promo2SinceWeek, Promo2SinceYear, PromoInterval):



Options considered: leave as-is vs. impute. Reasoning: directly verified all 544 nulls occur exactly where Promo2 == 0 — proven structural, not missing. Chosen approach: leave as NaN, no transformation. Downstream impact: none; later feature engineering can safely treat these as "not applicable."







Decision 2 — CompetitionDistance nulls (3 rows: stores 291, 622, 879):



Options considered: leave as NaN vs. impute a large placeholder value now for tree-model compatibility. Reasoning: at 0.3% of rows, this is a modeling-strategy decision (specific to model family in Stage 3/4), not a Stage 1 factual-cleaning decision — imputing now would assert an assumption before the model architecture is even chosen. Chosen approach: leave as NaN in Stage 1; revisit specific imputation strategy in EDA/modeling stages. Explicitly labeled as deferred, not resolved.







Decision 3 — CompetitionOpenSinceMonth/Year nulls (354 rows):



Options considered: leave nulls only vs. add explicit "known/unknown" boolean flag. Reasoning: raw nulls represent genuine missingness (competitor exists per CompetitionDistance, but opening date unknown) — fabricating a date would misrepresent fact as fiction. Chosen approach: added competition\_open\_date\_known flag column; raw date fields left untouched as NaN. Validated: flag value counts (354 unknown / 761 known) exactly match null pattern, 0 mismatches.







Decision: Merge train.csv with store.csv on Store using a left join.



Validation performed: (1) merged row count equals train.csv row count (1,017,209), confirming no rows added/dropped; (2) StoreType/Assortment show 0 new nulls, confirming all store IDs matched; (3) CompetitionDistance nulls = 2,642, exactly matching the total daily rows for stores 291, 622, 879 — confirming nulls are the expected broadcast of the 3 known null stores, not a merge error.



Downstream impact: merged dataset now carries store-level context on every transactional row; the 3 stores with unknown CompetitionDistance remain explicitly unknown (not imputed), consistent with the earlier Stage 1 decision to defer that choice to modeling stages.







\## EDA evidence — CompetitionDistance missingness







Stores 291, 622, and 879 have unknown CompetitionDistance. Their profiles are heterogeneous: stores 291 and 879 share StoreType d / Assortment a but have materially different average sales, while store 622 is StoreType a / Assortment c and has below-segment-average sales.







Implication for Stage 3/4: If the selected model requires a numeric CompetitionDistance, evaluate segment-level median imputation (StoreType × Assortment) together with an explicit CompetitionDistance-missing indicator. Do not treat the imputed value as observed ground truth.







\## Correction: Closure-gap pattern does not match Stage 1 note



The Stage 1 note describing "180 stores, 184-day contiguous closure gap" is not supported by this dataset. Verified: only 2 stores (103, 1081) show a long contiguous closure, both spanning exactly 2013-01-01 to 2013-07-04 (185 days, not 184), left-censored at the dataset's start (no pre-closure data exists). Treat the original 180-store figure as unverified for this dataset version. See eda\_findings.md for full characterization.



Decision: Impute CompetitionDistance for stores 291, 622, 879 with StoreType×Assortment median plus a missing flag.



Options considered: (A) global median, (B) segment median + flag, (C) large placeholder, (D) leave NaN, (E) drop stores, (F) model-based fill.



Reasoning: Segment peers are large (d/a n=126, a/c n=220). Distance varies by format. The test set contains these stores, so dropping them is not possible. A placeholder misrepresents "unknown" as "no competitor".



Chosen approach: Option B, applied identically to train and test.



Labeled as assumption, not fact: The filled values are approximations; the flag marks them as such.



Downstream impact: Tree models can use the flag to separate filled values from observed ones. Resolves the Stage 1/2 deferral.



Decision: Use a single log1p-transformed CompetitionDistance column for all models.



Options considered: (A) one log column for all models, (B) raw and log both, (C) model-specific pipelines.



Reasoning: log1p is neutral for trees and necessary for linear regression. One column keeps a single shared feature table and avoids duplicated, correlated features.



Chosen approach: Option A. Raw column excluded from the model feature list.



Downstream impact: Linear coefficients are on the log scale.



Decision: No separate December flag; Month only.



Options considered: (A) Month only, (B) Month + IsDecember, (C) Month + Nov–Dec flag, (D) cyclical sin/cos.



Reasoning: Trees isolate month 12 with one split. The linear pipeline one-hot encodes Month, giving December its own coefficient. A Nov–Dec flag is not supported by EDA (November 2014 is confounded with a promo spike). Cyclical encoding would blur December and January, which are opposite extremes. The test set (Aug–Sep 2015) does not contain December.



Chosen approach: Option A. Month is stored as 1–12; linear pipeline applies one-hot.



Downstream impact: Verify in 3B.4 that each model's predicted monthly means reproduce the December peak and January low. Add IsDecember only if validation shows the models miss it.



Decision: Promo × StoreType interaction built only in the linear pipeline.



Options considered: (A) Promo only, (B) interaction columns in the saved table, (C) interaction in the linear pipeline only, (D) separate model per StoreType.



Reasoning: Trees learn the interaction through splits. Linear needs explicit terms. One raw table keeps comparisons fair. Type b has 17 stores, too few for its own model.



Chosen approach: Option C.



Labeled as assumption, not fact: Promo is known in advance at prediction time.



Downstream impact: Implement in 3B.3. Set tree depth high enough to learn the interaction.



Decision: Define Promo2 features as Promo2\_enrolled and Promo2\_round\_start\_month.



Options considered: (A) enrolled flag, (B) round-start-month flag, (C) months-since-round-start, (D) raw columns only.



Reasoning: Documentation gives round start months but not round length, so a true "active now" flag would rely on an unverifiable assumption. A and B need no duration assumption. Raw nulls break linear and random forest. A date-aware flag avoids marking pre-start days as Promo2.



Chosen approach: A and B; C held as a Part B experiment.



Labeled as assumption, not fact: ISO week approximates Rossmann's week numbering.



Downstream impact: Both features are 0 for non-Promo2 stores. In test, enrolled equals Promo2.



Decision: Fixed 48-day forecast horizon. All lag and rolling features use sales at least 48 days old.



Options considered: (A) fixed 48-day horizon, (B) recursive, (C) next-day lags, (D) hybrid by horizon bucket.



Reasoning: Test spans 48 days (2015-08-01 to 2015-09-17) with no Sales, so short lags do not exist for most of it. Next-day lags would give an optimistic validation score that does not carry to the real task. Recursive forecasting compounds errors and adds complexity.



Chosen approach: Option A. Lags are multiples of 7 and at least 49 days so weekday stays aligned.



Downstream impact: Naive baseline in 3B.1 must be horizon-aware (cannot use last week). Validation windows are 48 days long to match.



Decision: Lag/rolling feature set = lag\_49, lag\_56, and a 28-day rolling mean shifted by 48 days.



Options considered: (1) lag\_49 only, (2) small set, (3) large set of lags and windows.



Reasoning: Lags give the same-weekday pattern; the rolling mean gives the store's recent level. EDA does not call for more, and a large set adds correlated columns and cold-start loss with only \~2.5 years of data.



Chosen approach: Option 2. More features only if validation shows a measurable gain.



Decision: Lag/rolling features = lag\_49, lag\_56, roll\_mean\_28 (28-day, shifted 48 days), on a fixed 48-day horizon, open days only.



Options considered: (1) lag\_49 only, (2) small set, (3) large set; (A) include closed days, (B) open days only; (A) drop cold start, (B) keep NaN, (C) fill with early mean; (A) fill blanks with rolling mean, (B) fill with 0, (C) leave NaN, (D) fill with previous same-weekday value.



Reasoning: Fixed 48-day horizon matches the test. Open-days-only respects the EDA's availability-vs-demand separation. Dropping cold start keeps the model comparison fair. The same-weekday fill (D) failed due to index alignment after four attempts; rolling-mean fill is the pragmatic fallback.



Chosen approach: Small set, open days only, drop cold start, rolling-mean fill with flags, store-mean fallback for 87 residual rows.



Labeled as assumption, not fact: ISO week approximates Rossmann's week numbering. The 180-store gap's cause is unknown.



Downstream impact: 99,820 rows dropped. 17% of remaining rows have filled lags, flagged. Correction notes added to decision\_log.md and eda\_findings.md.



Correction: The Stage 2 claim that "the 180-store, 184-day gap does not exist" is wrong for missing rows. Verified: 180 stores are missing 184 days of rows (2014-07-01 to 2014-12-31), plus one store missing one day. Stage 2 only searched Open==0 rows and could not detect absent rows. The original Stage 1 note was correct about absent rows. Stores 103 and 1081 remain a separate, real finding (Open==0 rows, 2013-01-01 to 2013-07-04).



Decision: Encode StoreType and Assortment as one combined Segment column (9 levels).



Options considered: (A) separate one-hot columns, (B) combined Segment column, (C) combined with sparse levels merged, (D) separate columns for trees and combined for linear.



Reasoning: Assortment b exists only within StoreType b (confirmed, 0 rows elsewhere), so separate columns would split one segment's effect across overlapping columns. A combined column mirrors the real structure and serves every model from one shared table. Merging the one-store b\_c level would be arbitrary, with no EDA support for the target level.



Chosen approach: Option B. Segment\_code from training segments for trees; one-hot inside the linear pipeline. StoreType kept for the Promo interaction only.



Labeled as assumption, not fact: b-segment coefficients describe 17 stores only; b\_c is a single store.



Downstream impact: Plain StoreType and Assortment stay out of the model feature list. Linear pipeline applies one-hot on Segment and builds the Promo × StoreType terms.

