# Transaction Data Cleaning & Association Rule Mining

A data-mining project that cleans a messy dataset of 100,000 retail transactions, then mines product association rules with the **Apriori** algorithm.

## Pipeline

**1. Data cleaning** (`dirty_financial_transactions.csv` → `transactions_cleaned.csv`)

- Drop rows without a `Transaction_ID` and remove duplicate IDs (100,000 → 94,040 rows)
- Fix quantities (negatives to positives, missing values to 1)
- Convert prices to numbers, fill values that don't parse with the median, and make negatives positive
- Standardise payment methods (`pay pal` → `paypal`) and statuses (`Complete` → `Completed`, missing → `Unknown`)
- Parse dates, with a fallback for invalid ones
- Normalise product names (`Tab` → `Tablet`)

**2. Association rules** (`transactions_cleaned.csv` → `association_rules.csv`)

- Group products by customer and one-hot encode them
- Find frequent itemsets with Apriori (`min_support = 0.1`)
- Keep rules with confidence ≥ 0.3 and report support, confidence and lift

### Known issue

6,640 raw prices are written with a currency sign, like `$420.21`. `pd.to_numeric` can't parse those, so the notebook treats them as missing and fills them with the median price (51.96), the same as the 33,497 prices that really are empty. The real values are lost. To keep them, strip the sign before converting:

```python
df_clean['Price'] = pd.to_numeric(df_clean['Price'].astype(str).str.replace('$', '', regex=False), errors='coerce')
```

Prices aren't used for rule mining, so `association_rules.csv` is unaffected.

## Files

| File | Description |
|---|---|
| `datamining (1).ipynb` | Notebook with the full pipeline (Google Colab) |
| `dirty_financial_transactions.csv` | Raw input (100,000 rows) |
| `transactions_cleaned.csv` | Cleaned output (94,040 rows) |
| `association_rules.csv` | Mined rules (180 rules) |
| `proj 2 report.pdf` | Project report |

## Run it

The notebook was written for Google Colab:

1. Open `datamining (1).ipynb` in [Google Colab](https://colab.research.google.com/).
2. Run the cells. When prompted, upload `dirty_financial_transactions.csv`.
3. The notebook writes `transactions_cleaned.csv` and `association_rules.csv`.

To run it locally instead, install `pandas numpy mlxtend` and replace the `google.colab` upload step with `pd.read_csv("dirty_financial_transactions.csv")`.

## Tech

pandas · NumPy · mlxtend (Apriori)

## License

[MIT](LICENSE)
