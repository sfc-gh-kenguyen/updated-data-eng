author: Gilberto Hernandez, Rida Safdar, Kevin Nguyen, Snowflake CoCo
id: snowflake-northstar-data-engineering
categories: snowflake-site:taxonomy/solution-center/certification/quickstart, snowflake-site:taxonomy/product/data-engineering, snowflake-site:taxonomy/product/ai, snowflake-site:taxonomy/feature/cortex-analyst, snowflake-site:taxonomy/feature/dynamic-tables, snowflake-site:taxonomy/feature/semantic-views, snowflake-site:taxonomy/use-case/data-engineering
language: en
summary: Build a fully prompt-driven I-T-D data pipeline in Snowflake using CoCo, Dynamic Tables, and a Cortex Agent accessed through Snowflake CoWork to discover why Hamburg sales dropped.
environments: web
status: Published
feedback link: https://github.com/Snowflake-Labs/sfguides/issues


# Getting Started – Data Engineering with Snowflake
<!-- ------------------------ -->
## Overview
Duration: 3

In this Quickstart, you'll build an end-to-end data pipeline in Snowflake using the **Ingestion–Transformation–Delivery** (I-T-D) framework — and you'll do it the modern way: by prompting **Snowflake CoCo** to write the SQL for you, inside a single **notebook** running in a **Git-backed Snowsight Workspace**.

**Ingestion**

We'll load data from:

- Snowflake Marketplace (live weather data share, installed programmatically)
- AWS S3 (Tasty Bytes sales CSVs via `COPY INTO`)

**Transformation**

We'll transform our data using:

- SQL User-Defined Functions (UDFs)
- **Dynamic Tables** (replacing static views with automatically-refreshing tables)

**Delivery**

We'll deliver a final data product using:

- A **Semantic View** that defines the business metrics analysts care about
- A **Cortex Agent** backed by Cortex Analyst
- **Snowflake CoWork** — the natural-language interface where analysts ask questions

### Prerequisites
- Some basic familiarity with SQL

### What You'll Learn
- The Ingestion-Transformation-Delivery (I-T-D) framework for data pipelines
- How to clone a public GitHub repo as a Git-backed Snowflake Workspace and run a notebook inside it
- How to connect a Snowflake Notebook to a runtime
- How to use **CoCo** to generate and run SQL from natural-language prompts
- How to load data from Snowflake Marketplace and AWS S3
- How to create **Dynamic Tables** that automatically refresh as source data changes
- How to write SQL UDFs and invoke them inside Dynamic Tables
- How to define a **Semantic View** with dimensions, metrics, and verified queries
- How to create a **Cortex Agent** backed by a Semantic View
- How to access the agent through **Snowflake CoWork** to answer business questions in natural language

### What You'll Need
- A free Snowflake trial account: [https://signup.snowflake.com/](https://signup.snowflake.com/?utm_source=snowflake-devrel&utm_medium=developer-guides&trial=student&cloud=aws&region=us-west-2&utm_campaign=introtosnowflake&utm_cta=developer-guides)
- The companion GitHub repo: [sfguide-snowflake-northstar-data-engineering](https://github.com/Snowflake-Labs/sfguide-snowflake-northstar-data-engineering)

### What You'll Build
- A fully prompt-driven I-T-D data pipeline in a single notebook
- A Cortex Agent that investigates *"What caused the Hamburg sales gap in February 2022?"*

<!-- ------------------------ -->
## Open a Snowflake Trial Account
Duration: 5

To complete this lab, you'll need a Snowflake account. A free Snowflake trial account will work just fine. To open one:

1. Navigate to [https://signup.snowflake.com/](https://signup.snowflake.com/?utm_source=snowflake-devrel&utm_medium=developer-guides&trial=student&cloud=aws&region=us-west-2&utm_campaign=introtosnowflake&utm_cta=developer-guides). This link pre-selects **AWS** and the **US West (Oregon)** region for you.

2. Complete the first page of the form.

3. On the next section, set the Snowflake edition to **Enterprise (Most popular)**.

4. Confirm **AWS – Amazon Web Services** is selected as the cloud provider (pre-filled by the link).

5. Confirm **US West (Oregon)** is selected as the region (pre-filled by the link).

6. Complete the rest of the form and click **Get started**.

![trial](./assets/trial.png)

<!-- ------------------------ -->
## Understand the Pipeline We'll Build
Duration: 3

Tasty Bytes is a food truck company that operates globally. You're a data engineer on the Tasty Bytes team. Data analysts recently flagged a troubling pattern:

> **Sales in Hamburg, Germany dropped to $0 for several days in February 2022.**

Your goal is to find out why — and to build an end-to-end pipeline that keeps analysts informed about Hamburg weather and sales in the future.

### The I-T-D Framework

Before diving in, it's worth understanding the framework we'll use: **Ingestion–Transformation–Delivery (I-T-D)**.

I-T-D is a standard pattern for structuring data pipelines. Instead of mixing raw data, business logic, and presentation into one place, it separates them into three distinct layers — each with a clear responsibility:

| Layer | Job |
|---|---|
| **Ingestion** | Land raw data exactly as it arrives, without modification |
| **Transformation** | Apply business logic, clean, enrich, and reshape the data |
| **Delivery** | Package the data as a consumable product for analysts or applications |

This separation matters because it makes pipelines easier to debug and extend. When something breaks, you know which layer to look at. When requirements change, you change the logic in one layer without touching the others.

### Here's the plan:

**Ingestion**

- Install a live weather data share from Snowflake Marketplace (Pelmorex) programmatically
- Load Tasty Bytes sales data from an AWS S3 bucket using `COPY INTO`

**Transformation**

- Create SQL UDFs to convert weather measurements to metric units
- Create four **Dynamic Tables** that automatically stay fresh as source data updates

**Delivery**

- Create a **Semantic View** joining sales and weather with business-friendly metric definitions
- Create a **Cortex Agent** backed by the Semantic View
- Register the agent in **Snowflake CoWork** and ask: *"What caused the Hamburg sales gap in February 2022?"*

The agent will surface the sales anomaly and correlate it with weather data.

Everything up through Transformation runs from a **single notebook** (`lab.ipynb`) that you'll clone from the companion repo. Delivery is done in the Snowsight UI. Let's get started!

<!-- ------------------------ -->
## Set Up Your Workspace and Notebook
Duration: 10

There are no setup scripts to run and no SQL worksheets to open first. Your very first action is to create a **Git-backed Workspace** that clones the companion repo — which contains just two files: the notebook you'll run (`lab.ipynb`) and a `README.md`.

### Sign in as ACCOUNTADMIN

Make sure you are signed into your trial account. Confirm your active role is **ACCOUNTADMIN**.

### Create a Git-backed Workspace

1. In Snowsight, navigate to **Projects » Workspaces**.

2. Click the **+** icon next to **Workspaces** → **Create Workspace from Git repository**.

3. Fill out the modal:
   - **Repository URL:** `https://github.com/sfc-gh-kenguyen/sfguide-snowflake-northstar-data-engineering`
   - **Workspace name:** anything you like (e.g., `northstar-data-eng`)
   - **API integration:** click **+ / Create new** and provide:
     - **Name:** `GITHUB_SFC_GH_KENGUYEN`
     - **API provider:** `git_https_api`
     - **Allowed prefixes:** `https://github.com/sfc-gh-kenguyen`
   - Check **Public repository**.

4. Click **Create**.

> **What just happened?** In the previous version of this lab you ran a `CREATE API INTEGRATION` statement by hand in a SQL worksheet. You no longer do that. The Workspace modal creates the same Git API integration for you from the values above — it's a one-time step that tells Snowflake which GitHub organization (`sfc-gh-kenguyen`) is an allowed source for Git-backed Workspaces. You'll never run that SQL yourself.

The Workspace opens with the repo's files visible in the file explorer on the left: `lab.ipynb` and `README.md`.

### Open the notebook and set its compute

1. In the Workspace file explorer, open **`lab.ipynb`**.

2. Set the notebook's active **role** and **warehouse** using the **role & warehouse picker at the top-left** of the Notebooks editor (you can also do this in a cell with `USE ROLE` / `USE WAREHOUSE`):
   - **Role:** **ACCOUNTADMIN**
   - **Warehouse:** **COMPUTE_WH** (the default warehouse in every trial account)

   This is the notebook's **query warehouse** — it runs every SQL cell and any Snowpark pushdown compute. (Rendering the interactive results grid doesn't consume credits.)

3. **Note on context:** notebooks in Workspaces do **not** automatically select a database or schema. This notebook handles that for you — its cells run `USE DATABASE` / `USE SCHEMA` and use fully qualified names (e.g., `TASTY_BYTES.RAW_POS.COUNTRY`) — so objects resolve no matter where you run it.

4. Work top-to-bottom through the notebook. Markdown cells explain each step; SQL cells contain the code (some of which you'll generate yourself with CoCo). Run a cell with **▶** (or **Shift+Enter**), or use **Run all** from the toolbar.

> The notebook session idle-suspends after about 30 minutes of inactivity. If it suspends, just restart the session (top-left) and re-run from where you left off — all objects you created persist in Snowflake.

> Throughout the lab, prompt cells show you the exact natural-language request to send to **CoCo**. Open the **CoCo** panel from the notebook toolbar, paste the prompt, and CoCo writes the SQL into a cell for you to run.

The first SQL cell in the notebook creates the `TASTY_BYTES` database and three schemas that map directly to the I-T-D pipeline layers:

- `RAW_POS` — raw ingested data (Ingestion)
- `HARMONIZED` — transformed and enriched data (Transformation)
- `ANALYTICS` — data products ready for consumption (Delivery)

…and grants the Cortex Agent and CoWork privileges your role needs. Run it before continuing.

<!-- ------------------------ -->
## Install the Weather Data From Snowflake Marketplace
Duration: 5

The first data source in our pipeline is live weather data. Snowflake Marketplace lets you mount a live dataset directly into your account without copying or moving any data — the provider keeps it fresh automatically.

Rather than clicking through the Marketplace UI, we install the listing **programmatically** from the notebook. This is the repeatable, scriptable way to acquire a Marketplace dataset: accept the listing's legal terms, then create a database directly from the listing.

Run this cell in the notebook:

```sql
-- As ACCOUNTADMIN
USE ROLE ACCOUNTADMIN;

-- Accept the listing terms programmatically
CALL SYSTEM$ACCEPT_LEGAL_TERMS('DATA_EXCHANGE_LISTING', 'GZSOZ1LLEL');

-- Install the Pelmorex Weather Source: Frostbyte listing
CREATE DATABASE IF NOT EXISTS FROSTBYTE_WEATHERSOURCE
  FROM LISTING 'GZSOZ1LLEL';
```

The share is now live in your account as `FROSTBYTE_WEATHERSOURCE`. No ingestion logic needed — the data is owned and refreshed by Pelmorex. Later transformation steps reference this database by name.

![data](./assets/weathersource.png)

<!-- ------------------------ -->
## Ingest Sales Data From S3
Duration: 20

Now let's load the Tasty Bytes sales data. It lives across many CSV files in a public AWS S3 bucket. We'll use Snowflake's `COPY INTO` command to bring it in.

### Run the boilerplate table DDL

The notebook's next cell contains `CREATE TABLE` statements for all the raw POS tables plus the `ANALYTICS.ORDERS_V` view. `ORDERS_V` is a denormalized flat join across the raw POS tables — it exists so that every downstream query has a single, simple source for sales data without needing to know the raw schema. This is boilerplate — there's nothing to "solve" here. Run the entire cell to create the empty target tables and the view.

### Open the CoCo panel

Open the **CoCo** chat panel from the notebook toolbar. Throughout this lab you'll copy prompts directly from the guide (and from the prompt cells in the notebook) and send them to CoCo. CoCo will write and run the SQL for you.

### STEP 1 — Create a CSV file format

Send this prompt to CoCo:

> *"Create a CSV file format named CSV_FF in TASTY_BYTES.PUBLIC with type = 'csv'."*

CoCo will write and run the SQL.

### STEP 2 — Create the external stage

Send this prompt to CoCo:

> *"Create an external stage named S3LOAD in TASTY_BYTES.PUBLIC that points to 's3://sfquickstarts/tastybytes/' and uses the CSV_FF file format."*

CoCo will write and run the SQL.

### STEP 3 — Load the COUNTRY table (the teaching example)

Send this prompt to CoCo:

> *"Write a COPY INTO statement that loads data from @tasty_bytes.public.s3load/raw_pos/country/ into TASTY_BYTES.RAW_POS.COUNTRY."*

You should see about 30 rows loaded successfully. **This single COPY INTO is the pattern for every table in the pipeline** — the stage path, the table name, and the `COPY INTO` verb are the only things that change.

### STEP 4 — Load all remaining tables (scale-up)

Send this prompt to CoCo:

> *"Load the remaining Tasty Bytes tables from the S3 stage @tasty_bytes.public.s3load into their corresponding tables in TASTY_BYTES.RAW_POS: FRANCHISE (raw_pos/franchise/), LOCATION (raw_pos/location/), MENU (raw_pos/menu/), TRUCK (raw_pos/truck/), ORDER_HEADER (raw_pos/order_header/), ORDER_DETAIL (raw_pos/order_detail/). Create a dedicated LARGE warehouse named LOAD_WH for the bulk load, switch to it, run the COPY INTOs, then drop LOAD_WH and switch back to COMPUTE_WH."*

CoCo will create a LARGE warehouse (`LOAD_WH`), run six `COPY INTO` statements, then drop `LOAD_WH` and restore `COMPUTE_WH`. This loads close to 1 GB of sales data. Wait for the success messages.

After all six loads complete, confirm the tables and their row counts in the object explorer on the left.

This completes the **Ingestion** stage of the pipeline.

<!-- ------------------------ -->
## Transform With UDFs and Dynamic Tables
Duration: 15

We have the raw data. Now we need to transform it into something analysts can query. In the original version of this pipeline, we used SQL views. This time, we're using **Dynamic Tables**.

A **Dynamic Table** is a table whose contents are defined by a query that Snowflake keeps up to date for you automatically. You write the `SELECT` once and declare a target freshness (`TARGET_LAG`); Snowflake works out the refresh schedule and, where possible, only reprocesses the rows that changed (incremental refresh). You get the readability of a view with the query performance of a table — and you never write or schedule pipeline code.

> **Are Dynamic Tables only for "non-real-time" analytics? No.** `TARGET_LAG` can be set as low as **1 minute** (or `DOWNSTREAM`, so a table refreshes just in time for the tables that depend on it), which makes Dynamic Tables a great fit for **both batch and near-real-time / low-latency analytics** — you tune freshness against cost by choosing the lag. Relax it to hours or days when data doesn't change often; tighten it when analysts need current data. (For the *lowest*-latency, high-concurrency serving — think real-time dashboards powering thousands of concurrent users, or data-powered APIs — Snowflake also offers [Interactive Tables](https://docs.snowflake.com/en/sql-reference/sql/create-interactive-table).) See the [Dynamic Tables overview](https://docs.snowflake.com/en/user-guide/dynamic-tables/overview).

> **Interoperability tip:** A Dynamic Table can also be a **Dynamic Iceberg Table** — it stores its results in open **Apache Iceberg** format on cloud storage so external engines like Spark and Trino can read the data directly, using the same refresh model. If your pipeline needs to feed a data lake or non-Snowflake engines, this is the option to reach for. See [Create a dynamic Apache Iceberg™ table](https://docs.snowflake.com/en/user-guide/dynamic-tables/create-iceberg).

The four Dynamic Tables in this pipeline build on each other in layers: `DAILY_WEATHER_DT` (base weather) feeds `WINDSPEED_HAMBURG_DT` and `WEATHER_HAMBURG_DT` (both Hamburg weather). `SALES_HAMBURG_DT` filters the sales data to Hamburg with a full date spine so zero-sales days are visible. `WEATHER_HAMBURG_DT` and `SALES_HAMBURG_DT` are the two tables that power the Semantic View.

### STEP 1 & 2 — Create the UDFs

A SQL UDF (User-Defined Function) is a reusable function you define once and call anywhere in SQL. The Pelmorex weather data uses imperial units, but analysts want metric. We'll create two UDFs — one to convert Fahrenheit to Celsius and one to convert inches to millimeters — that we'll invoke directly inside Dynamic Table queries.

> **What makes it a UDF:** a SQL UDF **must return a value** of the type declared in its `RETURNS` clause. Its body is a single expression whose result is returned every time you call the function. That's exactly what makes UDFs so handy for **data transformations** — encapsulate a unit conversion, formatting rule, or business calculation once, then reuse it everywhere in SQL, including inside Dynamic Table definitions like the ones below.

Send each prompt to CoCo:

**STEP 1:**
> *"Create a SQL UDF named FAHRENHEIT_TO_CELSIUS in TASTY_BYTES.ANALYTICS that accepts a NUMBER(35,4) parameter TEMP_F and returns the Celsius equivalent as NUMBER(35,4)."*

**STEP 2:**
> *"Create a SQL UDF named INCH_TO_MILLIMETER in TASTY_BYTES.ANALYTICS that accepts a NUMBER(35,4) parameter INCH and returns the millimeter equivalent as NUMBER(35,4)."*

Confirm they appear in `TASTY_BYTES.ANALYTICS` in the object explorer after CoCo runs each one.

### STEP 3 — DAILY_WEATHER_DT (full-refresh Dynamic Table)

This is the base weather Dynamic Table. It joins the live Pelmorex share with Hamburg postal codes to produce one row per postal code per day with city and country labels.

> **Key concept — why `REFRESH_MODE = FULL`?** The source here is the `FROSTBYTE_WEATHERSOURCE` share — a database we don't own. Snowflake can only enable the change tracking that powers *incremental* refresh on objects you own. For third-party shares, incremental refresh isn't possible, so we use `REFRESH_MODE = FULL` (Snowflake recomputes the whole table each cycle).

Run this in the notebook:

```sql
CREATE OR REPLACE DYNAMIC TABLE TASTY_BYTES.HARMONIZED.DAILY_WEATHER_DT
  TARGET_LAG = '1 day'
  REFRESH_MODE = FULL
  INITIALIZE = ON_CREATE
  WAREHOUSE = COMPUTE_WH
AS
SELECT
    hd.*,
    TO_VARCHAR(hd.date_valid_std, 'YYYY-MM') AS yyyy_mm,
    pc.city_name AS city,
    'Germany' AS country_desc
FROM FROSTBYTE_WEATHERSOURCE.onpoint_id.history_day hd
JOIN FROSTBYTE_WEATHERSOURCE.onpoint_id.postal_codes pc
    ON pc.postal_code = hd.postal_code
    AND pc.country = hd.country
WHERE pc.city_name = 'Hamburg';
```

> This refresh takes a few minutes. The downstream Dynamic Tables will pick it up automatically once it completes.

### STEP 4 — WINDSPEED_HAMBURG_DT

This Dynamic Table filters `DAILY_WEATHER_DT` to Hamburg specifically, tracking daily maximum wind speed. It's the intermediate table that isolates Hamburg's weather pattern — critical context for understanding why sales dropped on specific days.

Run this in the notebook:

```sql
CREATE OR REPLACE DYNAMIC TABLE TASTY_BYTES.HARMONIZED.WINDSPEED_HAMBURG_DT
  TARGET_LAG = '1 day'
  WAREHOUSE = COMPUTE_WH
AS
SELECT
    country_desc,
    city,
    date_valid_std,
    MAX(max_wind_speed_100m_mph) AS max_wind_speed_100m_mph
FROM TASTY_BYTES.HARMONIZED.DAILY_WEATHER_DT
WHERE city = 'Hamburg'
GROUP BY country_desc, city, date_valid_std
ORDER BY date_valid_std;
```

> **Expected message — this is not an error.** When you create `WINDSPEED_HAMBURG_DT` (and `WEATHER_HAMBURG_DT` below), you may see:
>
> *"Dynamic table … successfully created. FULL refresh mode was selected because: Change tracking is not supported on dynamic tables with 'FULL' REFRESH_MODE unless the Dynamic Table has FROZEN WHERE constraint specified."*
>
> Here's what it means: these tables read from `DAILY_WEATHER_DT`, which is a **FULL-refresh** table (because *it* reads a third-party share). A Dynamic Table that depends on a FULL-refresh source can't do incremental refresh either — change tracking isn't available up the chain — so Snowflake automatically selects FULL refresh for it too, unless you pin the rows with a `FROZEN WHERE` constraint. The word "successfully" is the important part: the table was created correctly. It simply refreshes in full each cycle rather than incrementally.

### STEP 5 — WEATHER_HAMBURG_DT (with metric conversions)

This is the final weather Dynamic Table — one of the two that power the Semantic View. It aggregates the postal-level data from `DAILY_WEATHER_DT` into **one row per date** and converts temperature and precipitation to metric units using the UDFs you just created.

Run this in the notebook:

```sql
CREATE OR REPLACE DYNAMIC TABLE TASTY_BYTES.HARMONIZED.WEATHER_HAMBURG_DT
  TARGET_LAG = '1 day'
  WAREHOUSE = COMPUTE_WH
AS
SELECT
    date_valid_std,
    MAX(TASTY_BYTES.ANALYTICS.FAHRENHEIT_TO_CELSIUS(avg_temperature_air_2m_f)) AS avg_temperature_celsius,
    MAX(TASTY_BYTES.ANALYTICS.INCH_TO_MILLIMETER(tot_precipitation_in)) AS avg_precipitation_mm,
    MAX(max_wind_speed_100m_mph) AS max_wind_speed_mph
FROM TASTY_BYTES.HARMONIZED.DAILY_WEATHER_DT
WHERE city = 'Hamburg'
GROUP BY date_valid_std;
```

### STEP 6 — SALES_HAMBURG_DT (Hamburg sales with date spine)

This is the sales-side counterpart to `WEATHER_HAMBURG_DT`. It filters sales to Hamburg and adds a **date spine** — a generated sequence of every calendar day — so that days with no orders still appear as rows in the data. Without the date spine, days with no activity would be absent entirely, making it impossible for the Semantic View to detect gaps in the sales record.

Run this in the notebook:

```sql
CREATE OR REPLACE DYNAMIC TABLE TASTY_BYTES.HARMONIZED.SALES_HAMBURG_DT
  TARGET_LAG = '1 day'
  WAREHOUSE = COMPUTE_WH
AS
WITH date_spine AS (
    SELECT DATEADD(DAY, SEQ4(), '2019-01-01') AS order_date
    FROM TABLE(GENERATOR(ROWCOUNT => 3000))
)
SELECT
    ds.order_date,
    ZEROIFNULL(SUM(o.price)) AS daily_sales,
    COUNT(o.order_id) AS num_orders
FROM date_spine ds
LEFT JOIN TASTY_BYTES.ANALYTICS.ORDERS_V o
    ON DATE(o.order_ts) = ds.order_date
    AND o.country = 'Germany'
    AND o.primary_city = 'Hamburg'
GROUP BY ds.order_date
ORDER BY ds.order_date;
```

After running, confirm `SALES_HAMBURG_DT` appears in `TASTY_BYTES.HARMONIZED`. The initial refresh runs in the background and may take a minute — wait before checking the preview.

### Summary

- `DAILY_WEATHER_DT` — weather for all Hamburg postal codes, full-refresh
- `WINDSPEED_HAMBURG_DT` — Hamburg wind speed over time
- `WEATHER_HAMBURG_DT` — Hamburg weather in metric units, one row per day
- `SALES_HAMBURG_DT` — Hamburg sales by day, including days with zero sales

This completes the **Transformation** stage of the pipeline.

<!-- ------------------------ -->
## Deliver With Cortex Agent and CoWork
Duration: 15

We have clean, refreshing data. Now we need to make it accessible to analysts — in natural language. We'll do this by creating a Semantic View, a Cortex Agent, and accessing both through **Snowflake CoWork**. We'll build these in the Snowsight UI (no SQL required for this stage).

### STEP 1 — Create the Semantic View

A Semantic View is a layer on top of tables that describes your data in **business terms**: which columns are dimensions (things you filter and group by), which are facts (raw values), and which are metrics (aggregated measures). It also defines how tables join together.

Cortex Analyst needs this layer because LLMs can't reliably generate SQL against arbitrary table and column names. They can, however, reason about business concepts — "daily sales," "windspeed," "city" — and translate questions like *"Why did sales drop?"* into correct SQL. The Semantic View is the bridge between natural language and your data model.

1. In Snowsight, navigate to **AI & ML → Analyst**.

2. Click **Create with Autopilot** in the top right.

3. Confirm your role is **ACCOUNTADMIN** and warehouse is **COMPUTE_WH**.

4. Click **Skip** on the **Provide context** page.

5. Configure the following:
   - **Name:** `HAMBURG_INSIGHTS_SV`
   - **Location to store:** `TASTY_BYTES.ANALYTICS`
   - **Select tables:** Select `TASTY_BYTES.HARMONIZED.SALES_HAMBURG_DT` and `TASTY_BYTES.HARMONIZED.WEATHER_HAMBURG_DT`
   - **Select columns:** Select all columns

6. Click **Create**.

7. In the semantic view editor, click **Edit** next to `SALES_HAMBURG_DT`. If Autopilot added a unique or primary key on `ORDER_DATE`, remove it and click **Save**.

   > **Why remove it?** `SALES_HAMBURG_DT` is the **"many"** side of the relationship we're about to define (many sales days roll up to one weather day per date). A primary/unique key asserts that the keyed column uniquely identifies each row and that this table is a *lookup* table. If Autopilot marks `ORDER_DATE` as a key, it can lead Cortex Analyst to treat the join as one-to-one and skip the aggregation we actually need — producing wrong totals. Removing the key keeps `SALES_HAMBURG_DT` correctly modeled as the many-side fact table.

8. Click **Edit** next to `WEATHER_HAMBURG_DT`. If it's not already there, click **+ Unique Key**, add `DATE_VALID_STD` as the unique key, and click **Save**. (Autopilot often detects this automatically — if `DATE_VALID_STD` is already listed as the unique key, you can leave it as-is.)

   > **Why this one *does* need a key:** `WEATHER_HAMBURG_DT` is the **"one"** side — exactly one weather row per date. Declaring `DATE_VALID_STD` as its unique key tells Analyst it's safe to attach a single day's weather to each sales day, which is what makes the many-to-one join below valid.

9. Scroll down and click **+** on **Relationships**. Configure the following:
   - **From Table:** `SALES_HAMBURG_DT`
   - **To Table:** `WEATHER_HAMBURG_DT`
   - **Relationship Type:** Many to One
   - **From Column:** `ORDER_DATE`
   - **To Column:** `DATE_VALID_STD`

   Click **Add** to add the relationship, then click **Save** in the top right to save the entire Semantic View.

   > **Why the relationship matters:** This is the single most important step in the delivery stage. The relationship tells Cortex Analyst *how sales and weather connect* — join each sales day (`ORDER_DATE`) to the matching weather day (`DATE_VALID_STD`). Without it, Analyst sees two unrelated tables and physically cannot answer any question that correlates them — including the one this whole lab is built around ("did the sales gap line up with a windspeed event?"). Declaring the join as **Many to One** (many sales days → one weather day) lets Analyst pull each day's weather metrics onto the sales timeline and reason about the two together.

### STEP 2 — Create the Cortex Agent

**Cortex Analyst** is Snowflake's text-to-SQL engine — it reads the Semantic View to understand your data model and converts natural-language questions into SQL queries. The **Cortex Agent** is the AI orchestration layer that receives questions from analysts and routes them to Cortex Analyst as the tool.

1. In Snowsight, navigate to **AI & ML → Agents**.

2. Click **Create agent** in the top right.

3. Configure:
   - **Database and schema:** `TASTY_BYTES.ANALYTICS`
   - **Agent object name:** `HAMBURG_AGENT`

4. Click **Create**.

5. Click **Configuration** near the top of the agent editor.

6. Under the **General** tab, set:
   - **Description:** `I am a Hamburg Sales & Weather Intelligence Agent. I analyze Tasty Bytes food truck sales in Hamburg, Germany alongside local weather data to help answer questions about why sales fluctuated.`
   - **Example questions:**
     - `What caused the Hamburg sales gap in February 2022?`
     - `What were Hamburg's best and worst sales months in 2022?`
     - `Is there a relationship between temperature and daily sales in Hamburg?`

7. Under the **Instructions** tab, set:
   - **Orchestration Instruction:** `Whenever you can answer visually with a chart, always choose to generate a chart even if the user didn't specify to.`
   - **Response instructions:** `Give concise, accurate answers about Tasty Bytes Hamburg sales and weather.`

8. Click **Tools → Add semantic view** and select **Add semantic view** from the options.

9. Configure the tool:
   - **Service database & schema:** `TASTY_BYTES.ANALYTICS`
   - **Select semantic view:** `HAMBURG_INSIGHTS_SV`
   - **Name:** `TASTY_BYTES_SALES_ANALYST`
   - **Description:** `Answers questions about Tasty Bytes Hamburg sales and weather`

10. Click **Add** then click **Save** in the top right.

### Ask the agent the key question

Since you created the agent through the UI, it is already available in Snowflake CoWork.

1. In Snowsight, navigate to **AI & ML → Snowflake CoWork**.

2. Select **HAMBURG_AGENT** from the agent list.

3. Ask:

   > *"What caused the Hamburg sales gap in February 2022?"*

The agent will query the Semantic View, retrieve the sales and weather data for February 2022, and render a chart showing the correlation between the sales gap and windspeed.

This completes the **Delivery** stage of the pipeline.

<!-- ------------------------ -->
## Conclusion And Resources
Duration: 3

Congratulations! You've successfully built a fully prompt-driven, end-to-end data pipeline in Snowflake using the **Ingestion–Transformation–Delivery** framework — all from a single notebook plus a few UI steps.

### What You Learned

**Ingestion**

- Installed a live weather dataset from Snowflake Marketplace programmatically (zero copy, provider-maintained)
- Loaded ~1 GB of Tasty Bytes sales data from AWS S3 using `COPY INTO`

**Transformation**

- Wrote SQL UDFs that convert imperial weather measurements to metric
- Created four **Dynamic Tables** that refresh automatically:
  - `DAILY_WEATHER_DT` with `REFRESH_MODE = FULL` (required for third-party shares)
  - `WINDSPEED_HAMBURG_DT` filtering to Hamburg
  - `WEATHER_HAMBURG_DT` with metric conversions via UDFs
  - `SALES_HAMBURG_DT` with a date spine to expose zero-sales days

**Delivery**

- Defined a **Semantic View** with business-friendly dimensions and metrics using the Analyst UI
- Created a **Cortex Agent** backed by Cortex Analyst using the Agents UI
- Accessed the agent through **Snowflake CoWork** and asked a natural-language question that produced a chart revealing the windspeed-sales correlation

**The pipeline surfaced the answer**: an analysis of Hamburg sales and weather data in February 2022 reveals a 6-day gap in sales that correlates with a significant windspeed event.

### Related Resources

- [Companion repo: sfguide-snowflake-northstar-data-engineering](https://github.com/Snowflake-Labs/sfguide-snowflake-northstar-data-engineering)
- [Git-backed Workspaces documentation](https://docs.snowflake.com/en/user-guide/ui-snowsight/workspaces-git)
- [Dynamic Tables documentation](https://docs.snowflake.com/en/user-guide/dynamic-tables/overview)
- [Create a dynamic Apache Iceberg™ table](https://docs.snowflake.com/en/user-guide/dynamic-tables/create-iceberg)
- [Semantic Views documentation](https://docs.snowflake.com/en/user-guide/views-semantic/sql)
- [Cortex Agents documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents-manage)
- [Snowflake CoWork documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/snowflake-cowork/getting-started)
- [Snowflake Documentation](https://docs.snowflake.com/)
- [Learn more at Snowflake Northstar for developers](/en/developers/northstar/)
