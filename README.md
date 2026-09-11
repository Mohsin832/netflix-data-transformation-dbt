# Netflix Data Transformation with dbt

An end-to-end analytics engineering project that takes raw Netflix titles and credits data from Kaggle, loads it into a cloud data warehouse, and transforms it into clean, tested, analytics-ready tables using **dbt**.

## What this project does

Raw Netflix data (titles and credits) is sourced from Kaggle, uploaded to **AWS S3**, staged and loaded into **Snowflake**, and then transformed through dbt's layered modeling approach — staging → intermediate → marts — with automated data quality tests at each stage.

The result is a set of clean, well-modeled tables ready for analytics, reporting, or dashboarding, built the way a production analytics engineering pipeline would be structured.

## Data flow

![Data flow diagram](flow%20diagram.png)

1. **Origin** — Netflix titles & credits CSVs are pulled from Kaggle.
2. **Loader** — Files are uploaded to an AWS S3 bucket, then loaded into a Snowflake external stage.
3. **Snowflake tables** — Staged data is copied into raw tables in Snowflake via `COPY INTO`.
4. **dbt transformations** — dbt reads from the raw tables and transforms them through three layers:
   - `staging/` — 1:1 with source tables; renames, type-casts, and light cleanup, no business logic.
   - `intermediate/` — joins titles to credits, reshapes cast/crew data, applies business logic.
   - `marts/` — final, analytics-ready tables built for reporting and downstream use.
5. **Tests** — dbt tests (uniqueness, not-null, referential integrity, and custom checks) validate the data at each layer.

## Project structure

```
.
├── analyses/          # Ad-hoc analytical queries (not part of the DAG)
├── macros/            # Reusable Jinja macros
├── models/
│   ├── staging/        # Cleaned, renamed source data
│   ├── intermediate/   # Joins and business logic
│   └── marts/           # Final analytics-ready tables
├── seeds/              # Static/reference CSV data loaded via dbt
├── snapshots/           # Slowly changing dimension snapshots
├── tests/               # Custom data tests
├── dbt_project.yml      # Project configuration
└── flow diagram.png     # Pipeline architecture diagram
```

## Tech stack

| Layer | Tool |
|---|---|
| Data source | Kaggle (Netflix titles & credits datasets) |
| Object storage | AWS S3 |
| Data warehouse | Snowflake |
| Transformation | dbt |
| Version control | GitHub |

## Prerequisites

- Python 3.8+
- A Snowflake account with a warehouse, database, and schema set up
- An AWS account with an S3 bucket for raw file storage
- dbt installed with the Snowflake adapter

```bash
pip install dbt-snowflake
```

## Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/Mohsin832/netflix-data-transformation-dbt.git
   cd netflix-data-transformation-dbt
   ```

2. **Download the dataset**
   Get the Netflix titles and credits CSVs from Kaggle and upload them to your S3 bucket.

3. **Load data into Snowflake**
   Create a Snowflake external stage pointing at your S3 bucket, then `COPY INTO` your raw tables:
   ```sql
   CREATE OR REPLACE STAGE netflix_stage
     URL = 's3://<your-bucket>/<path>/'
     CREDENTIALS = (AWS_KEY_ID = '<key>' AWS_SECRET_KEY = '<secret>');

   COPY INTO raw.netflix_titles
   FROM @netflix_stage/titles.csv
   FILE_FORMAT = (TYPE = CSV SKIP_HEADER = 1);
   ```

4. **Configure your dbt profile**
   Add a profile for this project in `~/.dbt/profiles.yml`:
   ```yaml
   netflix_data_transformation_dbt:
     target: dev
     outputs:
       dev:
         type: snowflake
         account: <your_account>
         user: <your_user>
         password: <your_password>
         role: <your_role>
         database: <your_database>
         warehouse: <your_warehouse>
         schema: <your_schema>
         threads: 4
   ```

5. **Install dbt package dependencies** (if any are used)
   ```bash
   dbt deps
   ```

6. **Run the models**
   ```bash
   dbt run
   ```

7. **Test the data**
   ```bash
   dbt test
   ```

8. **Generate and view documentation**
   ```bash
   dbt docs generate
   dbt docs serve
   ```

## Model layers

- **Staging** — one model per source table, standardized column names and types, no joins.
- **Intermediate** — combines staged models (e.g. joining titles with their cast and crew), applies transformations that don't belong in a final mart.
- **Marts** — the tables analysts and BI tools query directly.

## Contact

Built by [Mohsin832](https://github.com/Mohsin832).
