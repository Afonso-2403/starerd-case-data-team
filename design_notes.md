# Design Notes — Survey Feedback Pipeline

## Tools & stack chosen
- **Pandas + Jupyter notebook** (single notebook: `src/data_processing.ipynb`) with separate cells for the different topics to facilitate discussion
- Aggregations are written as CSVs to the `aggregations/` directory

---

## High-level data flow
raw (CSV files: `data/`) → staging (`stg_survey_results`, `stg_user_metadata`) → cleaned → final fact table (`fct_survey_feedback`) → aggregations (CSV files in `aggregations/`)

---

## Transformation logic
- stg_survey_results
  - Remove exact duplicate submissions
  - Coerce `rating` to numeric; values not in 1..5 become null. (Having some trouble with this null)
  - Parse `timestamp` to datetime and store it in a new `submitted_at` column; invalid parses become null
  - Normalize `user_email` (trim + lowercase)
  - Flag data-quality issues (missing email, invalid timestamp, invalid rating)
  - Current notebook drops rows with nulls during staging for simplicity and given their small amount(would like to discuss other approaches)

- stg_user_metadata
  - Standardize `department` (Title Case)
  - Standardize `country` (Title Case) and flag missing values
  - Normalize `user_email` (trim + lowercase)
  - Create column that is a concatenation of name, country and department to only flag different emails for different combinations of these attributes
  - In this data there are no combination of those attributes that have different emails, didn't consider this further but would like to discuss possible approaches
  - Drop rows with nulls.

- fact / analytics
  - Join survey staging to user metadata on `user_email` (left join then filter to rows with metadata present for final analytics table in the notebook)
  - Dropped the intermediate columns as I assume they would be relevant only on the staging area
  - Computed the aggregations with pandas groupby statements; These are saved to CSV under `aggregations`
  - Given more context on the downstream needs of this table derived fields could be added (qualitative conversion of rating)

---

## Data quality & validation
- Not knowing how the survey is conducted, I would think that it is possible to have many of these validations on the client side when the respondent is answering the survey (rating being selected from a 1 to 5 scale, mandatory fields, etc). This way the raw data for this notebook would be "guaranteed" to be well behaved
- Validations implemented in notebook are described in the above section
- How to handle/monitor in production
  - Emit counts & metrics (rows processed, duplicates removed, counts of each data-quality flag) to a metrics store
  - Assuming validations at survey filling time, I would start with a simple approach of only logging missing data and have alerts if the volume is substantial. In that case a more complex approach would have to be considered, would like to discuss possible approaches

---

## Scalability & performance considerations
- If volume grew 100x:
  - The tables could be partitioned by date and/or region
  - Since the survey data does not expect updates or deletes, the data could be loaded incrementally, for example by week/day/hour depending on volume and requirements
  - The aggregations could also be computed incrementally (rolling aggregates)
  - I assume parallelism / chunked ingestion could be relevant but I have no experience with this, would like to hear about possible approaches
  - The `user_metadata` table could be divided into more tables, for example country and department dimension tables

---

## If I had more time…
- I would write this in a more software like fashion in python files with atomic methods
- Add automated tests (unit tests for transformation functions, dummy data for tests of the data functions).
- Add monitoring / dashboards for data-quality metrics and alerting.
- Integrate in the existing (create if non-existent) CI/CD flow

---
