# Snowflake Northstar: Data Engineering with Snowflake

Companion repo for the **[Getting Started – Data Engineering with Snowflake](https://quickstarts.snowflake.com/guide/snowflake-northstar-data-engineering)** Quickstart.

In this hands-on lab you build a complete, **prompt-driven** data pipeline for **Tasty Bytes** — a global food-truck company — to answer a real question from the analytics team:

> **Why did Tasty Bytes' sales in Hamburg, Germany drop to $0 for several days in February 2022?**

You'll build the pipeline with the **Ingestion → Transformation → Delivery (I-T-D)** framework, generating most of the SQL by prompting **Snowflake CoCo** in natural language — all from a single notebook running inside a Git-backed Snowsight Workspace.

## What's in this repo

| File | What it is |
|---|---|
| **`lab.ipynb`** | The notebook you run. It walks through Ingestion and Transformation cell-by-cell, with CoCo prompts shown inline. |
| **`README.md`** | This file. |

There are no setup scripts and no separate `project/` or `solution/` folders — everything runnable lives in `lab.ipynb`.

## What you'll build

**Ingestion**
- Install a live weather share (Pelmorex Weather Source: Frostbyte) from Snowflake Marketplace **programmatically** (`SYSTEM$ACCEPT_LEGAL_TERMS` + `CREATE DATABASE … FROM LISTING`).
- Load ~1 GB of Tasty Bytes POS data from a public **AWS S3** bucket with `COPY INTO`, scaling up on a temporary LARGE warehouse.

**Transformation**
- Two SQL **UDFs** that convert imperial weather measures to metric.
- Four **Dynamic Tables** that refresh automatically:
  - `DAILY_WEATHER_DT` — base Hamburg weather (`REFRESH_MODE = FULL`, required for a third-party share)
  - `WINDSPEED_HAMBURG_DT` — Hamburg daily max wind speed
  - `WEATHER_HAMBURG_DT` — Hamburg weather in metric units, one row per day
  - `SALES_HAMBURG_DT` — Hamburg daily sales with a **date spine** so zero-sales days are visible

**Delivery** (done in the Snowsight UI — see the guide)
- A **Semantic View** joining Hamburg sales and weather.
- A **Cortex Agent** backed by the Semantic View.
- **Snowflake CoWork**, where you ask *"What caused the Hamburg sales gap in February 2022?"* and get a chart that surfaces the windspeed–sales correlation.

## Data layers

The pipeline uses a three-schema layout that maps directly to the I-T-D layers:

| Schema | Layer | Contents |
|---|---|---|
| `TASTY_BYTES.RAW_POS` | Ingestion | Raw POS tables loaded from S3 |
| `TASTY_BYTES.HARMONIZED` | Transformation | The four Dynamic Tables |
| `TASTY_BYTES.ANALYTICS` | Delivery | `ORDERS_V`, UDFs, and the Semantic View |

## Prerequisites

- A free **Snowflake trial account** — [sign up here](https://signup.snowflake.com/?utm_source=snowflake-devrel&utm_medium=developer-guides&utm_campaign=northstar-data-eng&utm_cta=developer-guides&trial=student&cloud=aws&region=us-west-2) (pre-selects AWS / US West Oregon).
- Some basic familiarity with SQL.

## How to run it

1. **Sign in** to Snowsight as **`ACCOUNTADMIN`**.
2. **Create a Git-backed Workspace**: *Projects » Workspaces » + » Create Workspace from Git repository*, using this repo's URL: `https://github.com/Snowflake-Labs/sfguide-snowflake-northstar-data-engineering`. When prompted, create a Git API integration (name `GITHUB_SNOWFLAKE_LABS`, provider `git_https_api`, allowed prefix `https://github.com/Snowflake-Labs`) and check **Public repository**. No SQL required — the modal creates the integration for you.
3. **Open `lab.ipynb`** and **connect it to a runtime**: set Role = `ACCOUNTADMIN`, Warehouse = `COMPUTE_WH`, then click **Connect / Start**.
4. **Run the cells top-to-bottom.** Where a markdown cell shows a CoCo prompt, open the **CoCo** panel, paste the prompt, and run the SQL it writes.
5. **Do the Delivery stage in the UI** by following the *"Deliver With Cortex Agent and CoWork"* section of the Quickstart guide.

## Resources

- [Quickstart guide](https://quickstarts.snowflake.com/guide/snowflake-northstar-data-engineering)
- [Dynamic Tables](https://docs.snowflake.com/en/user-guide/dynamic-tables/overview) · [Dynamic Apache Iceberg™ tables](https://docs.snowflake.com/en/user-guide/dynamic-tables/create-iceberg)
- [Git-backed Workspaces](https://docs.snowflake.com/en/user-guide/ui-snowsight/workspaces-git)
- [Semantic Views](https://docs.snowflake.com/en/user-guide/views-semantic/sql) · [Cortex Agents](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents-manage) · [Snowflake CoWork](https://docs.snowflake.com/en/user-guide/snowflake-cortex/snowflake-cowork/getting-started)
- [Snowflake Northstar for developers](https://www.snowflake.com/en/developers/northstar/)
