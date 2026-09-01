Decision: Keep the 54 rows where Open == 1 and Sales == 0 (and Customers == 0) unmodified.

Options considered: (A) leave as-is, (B) drop rows, (C) add an anomaly-flag column.

Reasoning: Rows are internally consistent (zero customers → zero sales, not contradictory), affect only \~0.005% of train.csv, and dropping/flagging would add complexity or create time-series gaps with no measurable benefit.

Chosen approach: Option A — no transformation.

Downstream impact: None expected; noted as an accepted rarity, not a resolved data error, since root cause (weather, disruption, etc.) can't be confirmed from data alone.

