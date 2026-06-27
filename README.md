# 🛍️ Dashdown Excel Connector Demo

A minimal, end-to-end example of building an interactive analytics dashboard from
a single **Excel workbook** with [Dashdown](https://github.com/DirendAI/dashdown) — no JavaScript,
no build step, just Markdown + SQL.

This repo is the worked example behind the tutorial
**[Getting Started with the Excel Connector in Dashdown](extra/blog-excel-connector-demo.md)**.
Every snippet in that post comes from the dashboard here.

## What it shows

Using the open-source **Super Store Sales 2015** workbook, the dashboard covers:

- KPI **counters** (orders, customers, sales, profit)
- **Bar / line / pie charts** (sales by category, margins, monthly trend, segments, regions)
- **Cross-sheet JOINs** — `Returns` and `Users` sheets joined back to `Orders` for
  returned-sales-by-category and a sales/profit-by-manager table
- An **interactive filtered table** (category / region / segment dropdowns)
- A **top-products** table
- **AI commentary** via `<Ask>` — an LLM read-out of the monthly trend

## Quick start

```bash
# Install Dashdown with Excel + Mistral (for <Ask>) support
pip install 'dashdown-md[excel,mistral]'

# The <Ask> component needs an LLM key, which the project reads at startup
export MISTRAL_API_KEY=...

# Run the dashboard
cd dashdown-excel-demo
dashdown serve .
# → http://127.0.0.1:8000
```

> **No API key?** Remove (or comment out) the `llm:` block in `dashdown.yaml` and
> the `<Ask>` tag in `pages/index.md`; everything else runs without a key.

## Project structure

```
dashdown-excel-demo/
├── dashdown.yaml                 # title, branding, theme
├── sources.yaml                  # the Excel connector
├── pages/
│   └── index.md                  # the dashboard
├── data/
│   └── excel/
│       └── SuperStoreUS-2015.xlsx
└── extra/                        # blog post (gitignored)
    └── blog-excel-connector-demo.md
```

## The connector

A single entry in `sources.yaml` turns the workbook into a queryable SQL database
(each sheet becomes a table, via DuckDB):

```yaml
superstore:
  type: excel
  path: data/excel/SuperStoreUS-2015.xlsx
  header: true
  sheets:
    - Orders
    - Returns
    - Users
```

## Useful commands

```bash
dashdown check                # validate the project
dashdown serve .              # dev server with live reload on http://localhost:8000
dashdown build . --out dist   # static export
dashdown components           # list available components

# Query the workbook directly
dashdown query 'SELECT "Product Category", SUM("Sales") FROM Orders GROUP BY 1' -c superstore
```

## Notes

- Column names with spaces use **double quotes** in SQL (`"Order Date"`); string
  values use single quotes.
- Filters are plain `${param}` references with an `'${param}' = ''` "all" guard —
  see `pages/index.md`.
- Data source: [Practice-Datasets-for-Excel](https://github.com/rohanmistry231/Practice-Datasets-for-Excel) (open source).

---

*Built with Dashdown — the Markdown analytics framework.*
