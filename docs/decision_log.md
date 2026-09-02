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

