\# DemandLens — Stage 2 EDA Findings



This document records evidence-based findings from Stage 2 exploratory data analysis. It is a living document and will be updated as remaining EDA investigations are completed.



\## Analysis principles



\- Sales comparisons use `Open == 1` unless the question is specifically about store availability or closures.

\- Fixed store-level attributes, such as `CompetitionDistance`, are analyzed at store level rather than by repeating the same value across daily rows.

\- Results describe observed associations. They do not establish causation unless the analysis design supports a causal claim.

\- Small groups are interpreted cautiously.



\## 1. Promo and sales



\*\*Question:\*\* Are sales higher when a promotion is active?



\*\*Method:\*\* Compared promo and non-promo days among open stores. Then compared each store's average sales on promo days with its own average sales on non-promo days.



\*\*Result:\*\* Promo days had higher sales than non-promo days in the pooled open-day comparison. The pattern remained after within-store comparison: the median store had a 40.6% higher average sales level on promo days, and the middle 50% of stores had promo lifts between 29.4% and 51.8%.



\*\*Interpretation:\*\* Promotions are strongly associated with higher sales for most stores. The within-store result shows that the relationship is not explained only by large stores receiving more promo days.



\*\*Caveat:\*\* This is still an association. Promo timing may overlap with weekday, holiday, or seasonal demand patterns.



\*\*Stage 3/4 implication:\*\* Treat `Promo` as a high-priority candidate forecasting feature. Test whether its effect varies by store segment.



\## 2. Promo lift by StoreType



\*\*Question:\*\* Does the promo-sales relationship differ by store format?



\*\*Method:\*\* Grouped per-store percentage promo lift by `StoreType`, then inspected the 17 StoreType-b stores individually. Cross-tabulated `StoreType` and `Assortment`.



\*\*Result:\*\* Median promo lift was 44.0% for StoreType a, 34.0% for c, and 36.5% for d. StoreType b had a lower median lift of 10.3%, but showed substantial store-to-store variation, ranging from -6.9% to 76.3%.



\*\*Interpretation:\*\* Promo responsiveness differs across store segments. StoreType a has the strongest typical response, while StoreType b is heterogeneous rather than uniformly unresponsive.



\*\*Caveat:\*\* StoreType b has only 17 stores. Assortment b appears almost exclusively within StoreType b, so StoreType and Assortment are confounded for this group. The lower type-b lift cannot be attributed to store format alone.



\*\*Stage 3/4 implication:\*\* Evaluate StoreType and Assortment interactions with `Promo` rather than assuming one common promo effect for every store.



\## 3. CompetitionDistance distribution



\*\*Question:\*\* What does the competitor-distance distribution look like before assessing its relationship with sales?



\*\*Method:\*\* Created one record per store and summarized non-null `CompetitionDistance` values.



\*\*Result:\*\* Competitor distance is heavily right-skewed: median distance is 2,325m, mean distance is 5,405m, and the maximum is 75,860m. Three stores have unknown CompetitionDistance, matching the Stage 1 finding.



\*\*Interpretation:\*\* A small group of very distant competitors pulls the mean upward. Raw distance is not well suited to a simple linear analysis without accounting for this skew.



\*\*Stage 3/4 implication:\*\* Consider a log representation of known competitor distance during feature evaluation; do not treat raw-distance linearity as an assumption.



\## 4. CompetitionDistance and sales



\*\*Question:\*\* Do stores with closer competitors sell less?



\*\*Method:\*\* Aggregated sales to store level and compared average sales with CompetitionDistance using Pearson and Spearman correlation on raw distance, plus Pearson correlation on log-transformed distance.



\*\*Result:\*\* Raw-distance correlations did not show a detectable relationship. After log transformation, there was a statistically detectable but small negative association between distance and average sales (r = -0.10, p < 0.001).



\*\*Interpretation:\*\* Competition distance may contain limited sales signal, but it explains very little average-sales variation by itself.



\*\*Caveat:\*\* This is not causal. Stores with different competitor distances may also differ by store format, location, traffic, or unobserved market conditions.



\*\*Stage 3/4 implication:\*\* Retain CompetitionDistance as a candidate feature, but do not expect it to be a dominant standalone predictor.



\## 5. CompetitionDistance by StoreType



\*\*Question:\*\* Is the distance-sales association consistent across store formats?



\*\*Method:\*\* Compared competitor-distance distributions by StoreType and repeated log-distance versus average-sales correlations within each StoreType.



\*\*Result:\*\* Typical competitor distance differs by StoreType. Median distance is 900m for type b, 1,660m for type c, 1,790m for type a, and 5,040m for type d. A small negative correlation between log-distance and average sales appeared only for StoreType a (r = -0.13, p = 0.001). No clear relationship was detected for types c or d; type b has too few stores for a reliable test.



\*\*Interpretation:\*\* Competition distance is not a consistent sales driver across store formats. The weak overall relationship is mainly associated with StoreType a.



\*\*Caveat:\*\* StoreType is related to competitor distance, so the overall association is confounded. It should not be interpreted as evidence that changing competitor distance would cause sales to change.



\*\*Stage 3/4 implication:\*\* Evaluate possible non-linear and StoreType-specific competition effects during modeling rather than relying on a single global linear relationship.



\## 6. Stores with unknown CompetitionDistance



\*\*Question:\*\* Are the three stores with unknown CompetitionDistance similar to their StoreType × Assortment peer groups?



\*\*Method:\*\* Compared average sales for stores 291, 622, and 879 with observed-distance stores in the same `StoreType` and `Assortment` segment.



\*\*Result:\*\* Stores 291 and 879 are both StoreType d / Assortment a but have very different average sales: store 291 is above the segment average, while store 879 is below it. Store 622 belongs to StoreType a / Assortment c and is also below its segment average.



\*\*Interpretation:\*\* The three unknown-distance stores are heterogeneous. StoreType and Assortment can provide context for a future estimated value, but they cannot recover the true distance.



\*\*Caveat:\*\* The Stage 1 decision remains unchanged: CompetitionDistance values are still unknown, not zero and not observed.



\*\*Stage 3/4 implication:\*\* If a selected model requires numeric CompetitionDistance, compare segment-level median imputation with an explicit `CompetitionDistance\_missing` indicator. Do not treat an imputed value as ground truth.



\## 7. StateHoliday and store availability



\*\*Question:\*\* How do state holidays affect whether stores are open?



\*\*Method:\*\* Compared total store-day observations, open store-day counts, and open rates across `StateHoliday` categories.



\*\*Result:\*\* On ordinary days, 85.5% of store-days are open. On public holidays, Easter holidays, and Christmas holidays, open rates fall to 3.4%, 2.2%, and 1.7% respectively.



\*\*Interpretation:\*\* State holidays are strongly associated with store closures. Most of the impact of a state holiday is operational availability, not only a change in demand among stores that trade.



\*\*Caveat:\*\* Ordinary-day closure also occurs because of weekly closures and the known closure-gap pattern.



\*\*Stage 3/4 implication:\*\* Keep `StateHoliday` available as a calendar feature and model it jointly with `Open`/store-availability logic rather than interpreting a holiday as only a sales-level signal.



\## 8. StateHoliday and open-day sales



\*\*Question:\*\* When stores remain open, do sales differ by state-holiday type?



\*\*Method:\*\* Restricted analysis to `Open == 1`, then compared sales summaries and distributions across `StateHoliday` categories.



\*\*Result:\*\* Open stores had higher median sales on state holidays than on ordinary open days. However, open-holiday samples are small: 694 public-holiday, 145 Easter-holiday, and 71 Christmas-holiday store-days, compared with 843,482 ordinary open store-days. Holiday sales distributions also have high variability and outliers.



\*\*Interpretation:\*\* Stores that remain open on state holidays tend to be higher selling, but this may reflect selective opening by stores in high-demand locations rather than a general holiday demand increase.



\*\*Caveat:\*\* This result is conditional on being open and is not evidence that a typical store would sell more if it opened on a state holiday.



\*\*Stage 3/4 implication:\*\* Treat holiday-related availability and holiday-related sales as separate mechanisms. Do not apply one global holiday uplift to all stores.



\## 9. StateHoliday effects by StoreType



\*\*Question:\*\* Do open-day state-holiday sales patterns differ by store format?



\*\*Method:\*\* Compared count, mean sales, and median sales by `StateHoliday × StoreType`, using only open store-days.



\*\*Result:\*\* Holiday sales patterns differ by store format. StoreType b remains high-selling when open on public, Easter, and Christmas holidays. StoreTypes a and d often show lower sales than their own ordinary-day levels. Several Easter and Christmas StoreType combinations have very small samples, including groups with fewer than 10 observations.



\*\*Interpretation:\*\* State-holiday effects are not uniform across store formats and likely reflect selective opening patterns.



\*\*Caveat:\*\* Sparse holiday × StoreType groups cannot support strong format-level conclusions, particularly for Easter and Christmas.



\*\*Stage 3/4 implication:\*\* Test holiday interactions with store characteristics if model validation shows they add predictive value; avoid assuming a uniform StateHoliday effect.



\## Remaining Stage 2 investigations



\- SchoolHoliday effects on open-day sales, including separation from StateHoliday dates.

\- Day-of-week demand patterns.

\- Monthly and annual seasonality, including a check for overlap with promo timing.

\- StoreType and Assortment profiles using sales, customers, and sales per customer.

\- Characterization of the known 180-store, 184-day closure-gap pattern: exact date windows, affected store segments, and pre/post behavior.

\- Final review and concise Stage 2 findings summary for later modeling stages.

