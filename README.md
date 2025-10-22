# Overview

This repository processes a sales dataset and publishes computed metrics as JSON via GitHub Pages.

What you get:
- A fixed execute.py that:
  - Works with Python 3.11+ and Pandas 2.3
  - Reads data.csv (preferred) or falls back to data.xlsx
  - Computes:
    - row_count
    - regions_count
    - top_n_products_by_revenue (n=3)
    - rolling_7d_revenue_by_region (latest 7‑day moving average per region)
  - Prints the result JSON to stdout (so CI can redirect to result.json)
- A GitHub Actions workflow at .github/workflows/ci.yml that:
  - Installs dependencies and runs ruff (lint results shown in CI logs)
  - Ensures data.csv exists (converts from data.xlsx if missing)
  - Runs: python execute.py > result.json
  - Publishes result.json using GitHub Pages
- data.csv must be committed and match the provided data.xlsx
- Do not commit result.json; it is generated in CI and served via Pages

# Setup

1) Python environment
- Create a virtual environment and install dependencies:
  - python -m venv .venv
  - . .venv/bin/activate  # On Windows: .venv\Scripts\activate
  - python -m pip install --upgrade pip
  - pip install "pandas==2.3.*" ruff

2) Files to add/update
- Replace/create execute.py with the corrected version from index.html
- Add the workflow at .github/workflows/ci.yml (content in index.html)

3) Convert and commit data.csv (must match data.xlsx)
- Option A (browser):
  - Open index.html from this repo locally
  - Drop data.xlsx into the converter
  - Download data.csv and place it at the repo root
- Option B (Python):
  - python -c "import pandas as pd; pd.read_excel('data.xlsx').to_csv('data.csv', index=False)"
- Commit the CSV:
  - git add data.csv
  - git commit -m "Add data.csv converted from data.xlsx"
  - git push

Note: The CI workflow also converts data.xlsx to data.csv if it is missing, but you should commit data.csv as required by the task.

# Usage

Local run:
- python execute.py > result.json
- cat result.json
- Do not commit result.json.

CI:
- On every push, GitHub Actions will:
  - Run ruff and show any lint messages in the logs
  - Convert data.xlsx to data.csv if needed
  - Run the script to generate result.json
  - Publish result.json via GitHub Pages
- After the workflow completes, visit the Pages URL in the job summary to access result.json.

Continuous Deployment:
- The .github/workflows/ci.yml provided:
  - Uses actions/setup-python to install Python 3.11
  - Installs pandas 2.3.*, openpyxl (for xlsx conversion), and ruff
  - Uploads result.json (and a minimal index.html) as a Pages artifact
  - Deploys with actions/deploy-pages

# Notes

- Ensure execute.py contains no "revenew" typo; it uses the correct "revenue" column throughout.
- The script prints JSON to stdout, enabling redirection to result.json in CI.
- The top N products are computed by total revenue and returned as a list of objects:
  - [{ "product": "...", "revenue": 123.45 }, ...]
- The rolling 7-day averages are computed on daily revenue per region with missing days filled as 0 to ensure a true 7-day window, and the latest value per region is emitted.