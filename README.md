# Global Al Adoption Workforce Displacement Index Analytics Challenge

## Onyx-DataDNA July 2026 · Power BI · DAX · Star Schema Modeling

A 3-page Power BI report analyzing how AI adoption reshaped employment across 30 countries, 25 industries, and 8 skill categories between 2021 and 2024, built for a policy coalition briefing on where reskilling investment should go next.

---

## Business Problem

A policy coalition needs to know three things before it allocates reskilling funding:
1. **How much did the arrival of generative AI (late 2022) actually change adoption?**
2. **Which industries and skill categories are most exposed and most underinvested?**
3. **Is job creation keeping pace with job displacement anywhere?**

---

## Dataset

| | |
|---|---|
| Grain | Country × Industry × Skill Category × Quarter |
| Records | 300 fact rows (0.3% fill rate of 96,000 possible combinations — sparse by design) |
| Period | 2021-Q1 → 2024-Q4 (16 quarters) |
| Dimensions | 30 countries, 25 industries, 8 skill categories |
| Source | Synthetic dataset, FP20 Analytics |

---

## Semantic Model

**Star schema**, 7 tables total:

```
dim_country ─┐
dim_industry ─┼──► fact_workforce_ai_index ◄── dim_date
dim_skill_category ─┘        ▲
                              │
                          Calendar (daily date table,
                          bridges to dim_date via
                          QuarterStartDate, bidirectional)
```

- **`_Measures`** dedicated disconnected table holding all 40+ DAX measures, separated from data tables for a clean field list
- **`Calendar`** daily calendar table (2021-01-01 → 2024-12-31), marked as the official Power BI date table since the native `dim_date` is quarterly grain and can't satisfy Power BI's contiguous-daily-date requirement
- All fact-to-dimension relationships verified with **zero orphaned foreign keys** and **zero duplicate grain** across all 300 records

### Calculated columns
- `Displacement Risk Tier` (Low/Medium/High/Critical, quartile bucketed)
- `Reskilling Duration Bucket` (Short/Medium/Long)
- `Mismatch Tier` + `Mismatch Tier Color` (rank-and-color pair, industry-level)
- `Job Creation Rank` + `Job Creation Rank Color` (rank-and-color pair, country-level)
- `Underpreparedness Flag Color` (calculated column + measure variant, for visual binding flexibility)

### Key measures
- `GenAI Era Adoption Lift` : pre/post generative-AI adoption comparison
- `QoQ Adoption Rate Change` : quarter-over-quarter trend
- `Risk-Investment Mismatch Score` : rank-sum priority scoring at industry level
- `Job Creation Coverage Ratio` : jobs created ÷ jobs displaced, by country
- `Reskilling $ per Job Displaced` : spend efficiency, by country

---

## Report Pages

1. **Global Adoption & the GenAI Inflection** : adoption trend, pre/post-GenAI comparison, developed vs. emerging economies
2. **Industry & Skill Risk Exposure** : underpreparedness scatter, risk-investment mismatch table, skill category duration vs. replaceability
3. **Country Outcomes & Reskilling Recommendations** : job creation coverage by country, spend-vs-displacement, findings panel

---

## Findings & Recommendations

### 1. The generative AI era changed adoption sharply, not gradually
Average AI adoption rate jumped from **20.4% (pre-GenAI) to 38.6% (post-GenAI)**, a **+18.2 point lift**. Almost the entire lift is concentrated in a single quarter: **2022-Q4 saw a +17.8 point quarter-over-quarter jump**, the largest single move across all 16 quarters observed. This wasn't a trend that accelerated, it was a step change coinciding exactly with generative AI tools going mainstream.

### 2. Developed vs. emerging economies are not where the real gap is
Adoption rate is nearly identical across development tiers, **30.98% (Developed) vs. 30.22% (Emerging)**, a gap of just **0.76 points**. Coalition messaging built around a "developed economies are pulling ahead" narrative isn't supported by this data. The real disparities live at the industry and country-spend level, not the tier level.

### 3. No country's job creation offsets its own displacement
Across all 30 countries, **not one** has a Job Creation Coverage Ratio above 1.0. The best performer, **United States, sits at 0.69** — meaning even the most favorable case only creates 69 jobs for every 100 displaced. Global Net Jobs Impact is **-290,430**. This is the single most important reframing for the briefing: **funding should be positioned as displacement mitigation, not displacement prevention**, no realistic level of job creation is currently closing this gap anywhere in the dataset.

### 4. Priority industries : risk high, investment low
Using a rank-sum Mismatch Score (combining displacement risk rank + reskilling investment rank), **7 of 25 industries (28%)** fall into the Critical Gap tier. The three most urgent:

| Rank | Industry | Mismatch Score |
|---|---|---|
| 1 | **Media & Entertainment** | 9 |
| 2 | **Financial Services** | 13 |
| 3 | **Legal Services** | 16 |

A separate lens : industries combining high automation susceptibility (>65) with low AI investment (<2.5% of revenue), flags a smaller, more concentrated group: **Manufacturing, Construction, Real Estate, and Public Health**. These four warrant particular attention since they combine structural exposure with under-preparation, not just a relative ranking gap.

### 5. Priority countries : underinvested relative to their own displacement
Ranked by reskilling dollars spent per job displaced (lower = more underinvested relative to the country's own problem, not relative to other countries' wealth):

| Rank | Country | $ per job displaced |
|---|---|---|
| 1 | **Indonesia** | $27.64 |
| 2 | **Egypt** | $28.01 |
| 3 | **Germany** | $33.50 |

Indonesia and Egypt fit the expected emerging-economy pattern. **Germany's presence on this list is the sharper finding**, a developed economy with clear fiscal capacity is still spending less per displaced worker than two emerging economies. This is worth flagging explicitly to the coalition rather than assuming underinvestment is purely a resource-constraint story.

### 6. Skill category risk is more nuanced than "some skills are just worse off"
Contrary to an initial assumption made mid-build (and corrected here), replaceability and reskilling duration run in **opposite directions** across skill categories in this dataset:

| Skill Category | Replaceability | Reskilling Duration |
|---|---|---|
| Manual Skilled Trades | 79.6 (highest) | 4.9 months (shortest) |
| Cognitive & Analytical | 20.9 (lowest) | 19.6 months (longest) |

The most-replaceable skill category is also the fastest to retrain; the least-replaceable is the slowest. There's no single skill category combining high risk with slow response capacity — the real reskilling bottleneck sits in the **middle tier** (e.g., Manual & Physical: moderate replaceability at 36.9, second-longest duration at 14.9 months), not at either extreme.

### Recommended reskilling reallocation logic
To keep the funding-priority methodology consistent across industries and countries (rather than using two different ad-hoc rankings), both use the same rank-sum approach:

```
Priority Score = Rank(Risk / Underinvestment, ascending) + Rank(Exposure Volume, descending)
```
Lower combined score = higher funding priority. This is already implemented as `Risk-Investment Mismatch Score` (industry level) and can be extended identically at the country level using `Reskilling $ per Job Displaced` + `Jobs Displaced` — a `Reallocation Priority Score` in the same spirit, built in the model but pending its own KPI card placement.

---

## Built With

- Power BI Desktop 
- DAX (RANKX, CALCULATE, context transition, rank-sum composite scoring)
  
---

## Known data caveats

- **Sparse fact table**: only 0.3% of possible country×industry×skill×quarter combinations are populated. Blank ≠ zero.
- **Two measures have ~3% null rows** (`avg_wage_change_pct`, `ai_tool_usage_hours_per_week`), `AVERAGE()` excludes these natively rather than coercing to zero.
- **`data_confidence_score` ranges from 0.34 to 0.999**, 80% of records sit at ≥0.7 confidence; high-confidence filtering is available as a report-level toggle.

---

