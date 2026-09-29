# IBM Data Analyst Capstone — Technology Trends Dashboard

Final project for the **IBM Data Analyst Professional Certificate** (Coursera), Module 5 — *Building a Dashboard* (Option A: IBM Cognos Analytics).

## Overview

This project builds an interactive 3-tab dashboard analyzing developer survey data (18,845 respondents, 114 columns — a subset of the Stack Overflow Developer Survey format used throughout the certificate). The dashboard covers:

- **Current Technology Usage** — languages, databases, platforms, and web frameworks developers currently use
- **Future Technology Trend** — the same four categories, but what developers *want* to work with
- **Demographics** — age, country, and formal education level of respondents

## Tools

- **Microsoft Excel / Power Query** — data reshaping (wide → long format)
- **IBM Cognos Analytics** — dashboard building and visualization

## The Technical Challenge

Several survey columns (e.g. `LanguageHaveWorkedWith`, `DatabaseWantToWorkWith`) store multiple values per respondent as a single semicolon-delimited string — for example, `"Python;JavaScript;SQL"`. To count and rank individual values (needed for every Top 10 chart in this dashboard), each value needs its own row — the same transformation `pandas.Series.str.split(';').explode()` performs in Python.

**IBM Cognos Analytics has no built-in equivalent of `.explode()`.** After ruling out every native option inside the tool itself (field properties, data formatting, the "Split Column" feature — which only splits into columns, never rows — calculated fields, and the fully manual table-union approach), the correct fix turned out to belong in the data-preparation layer, not the BI layer.

**Solution:** each multi-value column was reshaped independently in Excel using Power Query's `Split Column → By Delimiter → Advanced options → Split into: Rows` — the true equivalent of `.explode()`, unlike Cognos's column-only split. Each column required its own query starting fresh from the raw data; chaining transformations across different multi-value columns on the same query produces a combinatorial explosion (one attempt reached 22.3 million rows instead of the expected ~116,000) rather than the correct independent expansion.

The reshaped dataset (8 exploded tables + 1 demographics table, 9 sheets total) was then uploaded to Cognos as a single data module, where Cognos's native **Top/Bottom filter** handles counting and ranking dynamically — so the Top 10 in every chart recalculates automatically if the underlying data changes, rather than being frozen as pre-computed values.

## Dashboard Structure

| Tab | Panel | Chart type | Metric |
|---|---|---|---|
| Current Technology Usage | 1 | Bar chart | Top 10 Languages Have Worked With |
| | 2 | Column chart | Top 10 Databases Have Worked With |
| | 3 | Word cloud | Platforms Have Worked With |
| | 4 | Hierarchy bubble chart | Top 10 Web Frameworks Have Worked With |
| Future Technology Trend | 1 | Bar chart | Top 10 Languages Want To Work With |
| | 2 | Column chart | Top 10 Databases Want To Work With |
| | 3 | Tree map | Platforms Want To Work With |
| | 4 | Hierarchy bubble chart | Top 10 Web Frameworks Want To Work With |
| Demographics | 1 | Pie chart | Respondent distribution by age |
| | 2 | Map chart | Respondent count by country |
| | 3 | Line chart | Respondent distribution by education level |
| | 4 | Stacked bar chart | Respondent count by age, classified by education level |

## Key Findings

- JavaScript, SQL, and HTML/CSS lead current language usage; the "want to work with" ranking is nearly identical, with Go gaining ground.
- PostgreSQL leads current and desired database usage.
- AWS dominates cloud platform usage by a wide margin, both currently and as the most desired platform.
- The 25–34 age group makes up the largest share of respondents (41.3%), and Bachelor's degree is the most common education level.

## Repository Contents

- `survey_data_.xlsx` — the Power Query–transformed dataset (9 sheets: 8 exploded multi-value tables + demographics) uploaded to Cognos
- `dashboard_export.pdf` — exported PDF of the finished Cognos dashboard (all 3 tabs)

## Author

Built as part of the IBM Data Analyst Professional Certificate capstone project.
