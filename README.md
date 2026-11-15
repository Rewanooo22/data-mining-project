# Transaction Data Cleaning and Association Rules

A data-mining coursework project that does two things:

1. It audits and cleans a deliberately "dirty" synthetic financial-transactions dataset of 100,000 rows.
2. It mines association rules between products with the Apriori algorithm (`mlxtend`).

All the work is in one Google Colab notebook. The raw data, the cleaned data and the generated rules are committed as CSV files.

## What the notebook does

`datamining (1).ipynb` contains one code cell, which runs these steps in order.

### 1. Load

Upload `dirty_financial_transactions.csv` through Colab (`google.colab.files.upload`) and read it with pandas. The dataset has 8 columns: `Transaction_ID`, `Transaction_Date`, `Customer_ID`, `Product_Name`, `Quantity`, `Price`, `Payment_Method`, `Transaction_Status`.

### 2. Data quality assessment

The notebook prints a report on the raw data. These figures come from the committed notebook output:

| Check | Result |
|---|---|
| Rows | 100,000 |
| Missing values (total) | 69,971: `Price` 33,497, `Transaction_Status` 16,679, `Quantity` 5,019, `Transaction_ID` 5,018, `Transaction_Date` 4,880, `Customer_ID` 4,878 |
| Duplicated `Transaction_ID` values | 5,959 (this count includes the missing IDs) |
| Negative quantities | 31,619 |
| Negative prices | 29,952 |
| Payment method spellings | 7 variants, e.g. `pay pal`, `PayPal`, `PayPal `, `creditcard`, `Credit Card` |
| Transaction status spellings | `Pending`, `completed`, `Completed`, `complete`, `Failed`, plus missing values |
| Unparseable dates | 68,261 (e.g. `2025-02-30`, `2023-13-01`) |

### 3. Cleaning

The cleaning steps exactly as implemented:

| Column | Operation |
|---|---|
| `Transaction_ID` | Drop rows where it is missing, then drop duplicates (keep the first occurrence) |
| `Quantity` | Convert to numeric (invalid values become NaN), take the absolute value, fill NaN with `1` |
| `Price` | Convert to numeric (invalid values become NaN), fill NaN with the column median, take the absolute value |
| `Payment_Method` | Lowercase and remove spaces, giving `creditcard`, `paypal` and `cash` |
| `Transaction_Status` | Fill missing values with `Unknown`, normalise the capitalisation, and map `Complete` to `Completed` |
| `Transaction_Date` | Parse with `pd.to_datetime(errors="coerce")` and fill unparseable dates with `2023-01-01` |
| `Product_Name` | Strip whitespace and map `Tab`/`tab` to `Tablet` |

According to the cleaning summary in the notebook output, the result is 94,040 rows (5,960 removed).

### 4. Association rule mining

- **Baskets:** rows are grouped by `Customer_ID`, and each customer's list of `Product_Name` values becomes one transaction. Rows with a missing `Customer_ID` are dropped by the `groupby`.
- **Encoding:** `mlxtend.preprocessing.TransactionEncoder` turns the baskets into a one-hot table.
- **Apriori:** `apriori(..., min_support=0.1, use_colnames=True)`.
- **Rules:** `association_rules(..., metric="confidence", min_threshold=0.3)`, sorted by lift in descending order.
- **Export:** the rules are saved with the columns `Antecedent`, `Consequent`, `Support (%)`, `Confidence (%)` and `Lift`.

### 5. Output

The notebook writes `transactions_cleaned.csv` and `association_rules.csv` to `/content/` in Colab.

## Results (from the committed CSV files)

- `transactions_cleaned.csv` has 94,040 rows and 8 columns.
- `association_rules.csv` contains **180 rules**:
  - support from 83.0% to 93.3%
  - confidence from 85.8% to 96.9%
  - **lift of 1.0 for every rule**

A lift of 1.0 means the antecedent and consequent co-occur no more often than independence would predict. Each basket combines all of a customer's purchases, and the data has only a handful of distinct products, so almost every customer basket contains almost every product.

## Tech stack

- Python 3 (Google Colab)
- pandas, NumPy
- mlxtend (`TransactionEncoder`, `apriori`, `association_rules`)

## Project structure

```text
.
├── datamining (1).ipynb               # Colab notebook: assessment, cleaning, Apriori, export
├── dirty_financial_transactions.csv   # Raw input dataset (100,000 rows)
├── transactions_cleaned.csv           # Cleaned dataset produced by the notebook (94,040 rows)
├── association_rules.csv              # Mined rules produced by the notebook (180 rules)
├── proj 2 report.pdf                  # Report section describing the data-quality issues
├── -                                  # Stray 1-byte file (a single newline), not used
├── LICENSE
└── README.md
```

## Getting started

### Option A: Google Colab (as written)

1. Open `datamining (1).ipynb` in Google Colab.
2. Run the cell. When prompted, upload `dirty_financial_transactions.csv`.
3. Download `transactions_cleaned.csv` and `association_rules.csv` from the Colab file browser (`/content/`).

The cell imports `mlxtend` before its `!pip install mlxtend` line. If `mlxtend` is not already installed in your runtime, install it first and then re-run the cell.

### Option B: Run locally

```bash
git clone https://github.com/Rewanooo22/transaction-data-cleaning-association-rules.git
cd transaction-data-cleaning-association-rules

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install pandas numpy mlxtend jupyter
jupyter notebook "datamining (1).ipynb"
```

The repository has no `requirements.txt`. The notebook also uses Colab-only code, so edit your local copy before running it:

- Replace the upload lines (`from google.colab import files`, `uploaded = files.upload()`, `filename = list(uploaded.keys())[0]`) with `filename = "dirty_financial_transactions.csv"`.
- Change the output paths `/content/transactions_cleaned.csv` and `/content/association_rules.csv` to local paths, e.g. `transactions_cleaned.csv` and `association_rules.csv`. This overwrites the committed copies.
- Optionally remove the `!pip install mlxtend` line, since the package is installed above.

## Limitations

These observations come from reading the code and the committed outputs:

- **Prices with a currency symbol** (e.g. `$420.21`) do not parse as numbers. They become NaN and are replaced by the median price, so their original values are lost rather than recovered.
- **Product names:** only `Tab`/`tab` is mapped. Many truncated names remain in `transactions_cleaned.csv` (e.g. `Lapt`, `Ta`, `Coffee Ma`, `Smartph`) and are treated as separate items during mining.
- **Dates:** unparseable dates are replaced with a fixed placeholder (`2023-01-01`) instead of being repaired. 64,194 rows in `transactions_cleaned.csv` have this date.
- **Signs:** negative quantities and prices are made positive with `abs()` rather than treated as returns or errors.
- **Baskets:** they are built per customer, not per transaction. Combined with the very small product catalogue, this produces near-universal itemsets and rules with a lift of 1.0.
- The notebook runs in a single cell and depends on Colab for input and output.

## Future improvements (ideas)

- Strip currency symbols before numeric conversion.
- Map truncated product names to their full names, e.g. by prefix matching against the known catalogue.
- Parse mixed date formats explicitly instead of using a placeholder date.
- Build baskets at a finer level (per transaction or per date) and filter rules by lift greater than 1.
- Split the notebook into cells and add a `requirements.txt`.

## License

Released under the [MIT License](LICENSE).
