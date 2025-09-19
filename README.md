LinkedIn Job Market Analysis (2023–2024)

An end-to-end data engineering & analytics project using a U.S. LinkedIn job-postings corpus (2023–2024). The work covers ingestion, cleaning, dimensional modeling in PostgreSQL, and SQL analyses to benchmark compensation, compare geography, and surface applicant-demand hot spots and skills.

🔎 Objectives

Build an analysis-ready warehouse from raw CSV extracts (postings + company + skills/benefits mappings).

Standardize compensation (normalize pay periods, compute midpoints, filter outliers) to make salaries comparable.

Design a star schema to support fast, reusable SQL for salary, location, and demand questions.

Deliver BI-ready views and example queries that stakeholders can plug into dashboards.

🗃️ Dataset (high level)

Source: public LinkedIn job-postings dataset with companion files for companies, skills, benefits, industries, etc.

Format: CSVs (postings plus ~10 complementary files).

Grain: one row = one job posting; companion tables enrich with company/skills/industry metadata.

🏗️ Architecture & Modeling

Landing → Staging → Warehouse pattern in PostgreSQL.

Tools: csvkit/xsv for sanity checks & column pruning; ipython-sql/psycopg2 for DB ops from Jupyter.

Star schema centered on postings (fact) with dimensions:

jobs_info (role, experience level, application meta, benefits/skills aggregates)

companies (name, size, industry, employee/follower counts, specialties)

jobs_location (location text, ZIP, derived state)

posting_time (hour/day/month/quarter keys for time series)

🧹 Data Preparation (key steps)

Unzip & flatten directory structure; drop high-variance text columns not needed for analytics.

CSV validation with csvclean -n; header/row diagnostics with csvstat; selective column keeps via xsv select.

Deduping small multi-industry/company artifacts via sort + awk '!seen[$1]++'.

Compensation standardization: convert HOURLY/WEEKLY/MONTHLY/BW pay to annual, compute normalized mid-salary; remove problematic columns after recompute.

Geo enrichment: map ZIP → state from a public ZIP database and attach to jobs_location.

Time dimension: generate posting_time keys from listed/expiry timestamps for month/quarter rollups.

🧯 Data Quality

Validations for NOT NULL/PK/FK where appropriate; filtered incomplete postings; range checks on salaries; duplicate checks on company IDs; documented assumptions and lineage inside the notebook/SQL comments.

📊 Core Analyses (sample SQL deliverables)

Compensation benchmarking: average/midpoint salaries by industry × position type (focus on IT subsectors; entry-level cut).

Geography: state-level compensation comparisons; general vs. SQL-tagged roles; monthly posting volumes.

Applicant demand: views & applies aggregated by industry; remote vs. on-site attention (views).

Company & industry lenses: top-paying companies/industries (USD, capped to remove outliers).

📦 Schema (warehouse)
-- Fact
postings (job_key FK, company_key FK, location_key FK,
          original_listed_time_key FK, expiry_time_key FK,
          views, applies, min_salary, max_salary, normalized_salary)

-- Dimensions
jobs_info (job_key PK, job_id, title, experience level, work type,
           pay period, application meta, benefits, skills, currency, etc.)
companies (company_key PK, company_id, name, size, industry, employees, followers, specialties, …)
jobs_location (location_key PK, location, zip_code, state)
posting_time (key PK, hour, day, month, quarter, …)


Each table is annotated with column comments for self-documentation.

🛠️ Tech Stack

PostgreSQL, Jupyter (ipython-sql/psycopg2), csvkit/xsv, pandas/matplotlib for quick visuals.

▶️ How to Run (suggested)

Create DB and connect from Jupyter (%sql postgresql://<user>@/finalp).

Load CSVs into staging tables using COPY.

Run the provided SQL DDL to create warehouse tables & comments.

Execute ETL/standardization SQL blocks (pay period normalization, mid-salary calc, geo/time enrichment).

Materialize BI views or run sample queries for salary, geography, and demand.

📈 Example Questions this answers

How do average entry-level salaries vary across IT industries and position types?

Which states offer the highest/lowest compensation for entry-level roles and for SQL-tagged jobs?

Which industries attract the most views/applies (i.e., strongest applicant demand)?

🚧 Known Caveats

Salary fields are sparsely populated in places; outlier handling and currency filters are applied.

Single-year window may over-represent short-term trends; use with context.

Some company/location fields contain placeholders that are standardized or excluded.
