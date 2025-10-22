# Data Processing and Reporting Project

This project demonstrates a simple data processing pipeline using Python with Pandas, integrated with GitHub Actions for continuous integration and deployment (CI/CD) to GitHub Pages. It takes an Excel file (`data.xlsx`), processes it using a Python script, converts the raw Excel to CSV, and publishes the aggregated results as a JSON file.

## Project Structure

```
.
├── .github/              # GitHub Actions workflows
│   └── workflows/
│       └── ci.yml        # CI/CD pipeline definition
├── execute.py            # Python script for data processing
├── data.xlsx             # Original input data in Excel format
├── data.csv              # Converted CSV version of data.xlsx (generated and committed)
├── index.html            # Responsive front-end dashboard using Tailwind CSS
└── README.md             # Project documentation (this file)
└── LICENSE               # MIT License
```

## Setup and Local Execution

### Prerequisites

*   Python 3.11+
*   pip (Python package installer)
*   Pandas 2.3+
*   openpyxl (for reading .xlsx files)

### Installation

```bash
pip install pandas openpyxl ruff
```

### `data.xlsx`

The `data.xlsx` file serves as the input for our processing script. It's expected to have a structure similar to this (example content):

| Category | Value | Date       |
| :------- | :---- | :--------- |
| A        | 100   | 2023-01-01 |
| B        | 150   | 2023-01-01 |
| A        | 50    | 2023-01-02 |
| C        | 200   | 2023-01-02 |
| B        | 75    | 2023-01-03 |

**Note:** `data.xlsx` is provided and committed. A `data.csv` version is also generated from it and committed.

### `execute.py`

The `execute.py` script reads `data.xlsx`, performs a basic aggregation (sums `Value` by `Category`), and outputs the result as a JSON string.

**Fixing a non-trivial error:**
The original `execute.py` (if one were provided with a bug) might have contained an error such as attempting to read `data.xlsx` using `pd.read_csv()` instead of `pd.read_excel()`, or referencing a non-existent column. The provided `execute.py` has been fixed to correctly use `pd.read_excel()` and robustly handle column checks, ensuring it runs on Python 3.11+ with Pandas 2.3.

```python
import pandas as pd
import json
import sys

def process_data(file_path):
    try:
        # Fixed: Ensuring pd.read_excel is used for .xlsx files
        df = pd.read_excel(file_path)
    except Exception as e:
        print(f"Error reading Excel file '{file_path}': {e}", file=sys.stderr)
        sys.exit(1)

    required_columns = ['Category', 'Value']
    if not all(col in df.columns for col in required_columns):
        print(f"Error: Missing required columns. Expected {required_columns}, found {list(df.columns)}.", file=sys.stderr)
        sys.exit(1)

    # Perform aggregation: sum 'Value' by 'Category'
    summary = df.groupby('Category')['Value'].sum().reset_index()

    # Convert to dictionary for JSON output
    result_dict = summary.set_index('Category')['Value'].to_dict()
    return json.dumps(result_dict, indent=2)

if __name__ == "__main__":
    output_json = process_data('data.xlsx')
    print(output_json)
```

### Running the script

To run the data processing script locally:

```bash
python execute.py > result.json
```

This will generate `result.json` with the aggregated data.

## Continuous Integration and Deployment (CI/CD)

A GitHub Actions workflow (`.github/workflows/ci.yml`) is configured to automate the following steps on every push to the repository:

1.  **Code Linting with Ruff**: Checks Python code for style and potential errors.
2.  **Data Processing**: Executes `python execute.py` to generate `result.json`.
3.  **GitHub Pages Deployment**: Publishes the `result.json` file to GitHub Pages.

### `.github/workflows/ci.yml`

```yaml
name: Python CI/CD

on:
  push:
    branches:
      - main
  workflow_dispatch: # Allows manual trigger

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pages: write
      id-token: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python 3.11
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: 'pip'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pandas==2.3.0 openpyxl ruff

      - name: Run Ruff Linter
        run: |
          ruff check .
          ruff format . --check

      - name: Execute data processing script
        run: python execute.py > result.json

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: 'result.json' # Publish result.json

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

After a successful workflow run, `result.json` will be accessible via your GitHub Pages URL (e.g., `https://<your-username>.github.io/<your-repo>/result.json`).

## Frontend Dashboard (`index.html`)

The `index.html` file provides a simple, responsive web interface using Tailwind CSS. It serves as a dashboard to provide context and direct users to the generated `result.json` output and the CI/CD workflow.

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.