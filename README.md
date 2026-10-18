# Supply Chain Operations Intelligence

Why are orders late, what drives losses, and what should management fix first?
An end-to-end analysis of 180,519 order items using Python, SQL, Excel, Power BI, and two AI features, each tested honestly.


## Business problem
A global retailer ships orders through four shipping modes. More than half arrive late, and about 1 in 5 order items loses money. Management needs to know the causes and the first fixes.

## Data
- **Source:** [DataCo Smart Supply Chain (Kaggle)](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis)
- **Size:** 180,519 order items, 53 columns, January 2015 – January 2018
- **Note:** public dataset, not a live company. See Limitations.

## Tools
Python (pandas), SQL (DuckDB), Excel (PivotTables), Power BI (DAX), Google Gemini API, scikit-learn

## Key findings
1. **Shipping mode drives lateness.** Late rate on shipped orders: First Class 100%, Second Class 79.8%, Same Day 47.9%, Standard Class 39.8%. Overall: 57.3%.
2. **Delivery promises are unrealistic.** First Class promises 1 day but always takes exactly 2. Second Class promises 2 days but averages 3.99, the same as Standard Class (4.00).
3. **Volume matters more than rate.** Standard Class has the lowest late rate but creates 41.4% of all late items. First and Second Class together create 54.1% of late items from 34.9% of volume.
4. **Region barely matters.** Late rates range from 54.7% to 60.1% across regions (Canada 51.6%, but only 907 items).
5. **Late deliveries do not drive losses.** Loss rate: 18.6% for on-time items vs 18.7% for late items. Average profit: 22.58 vs 21.62.
6. **Losses are flat everywhere.** About 18.7% of items lose money regardless of discount level (18.1–19.6%) or category (16.8–19.6%).
7. **Profit depends on product mix.** Fishing earns 43.6 per item; Men's Footwear earns 14.0.
8. **Lateness never improved.** Monthly late rate stayed at 55–59% for 3 years. Order volume dropped about 60% from late 2017.

## Recommendations
1. **Align First and Second Class promises with real delivery times**, or speed them up. This targets 54% of late items. Note: changing promises makes expectations honest; it does not make deliveries faster.
2. **Reduce Standard Class variability.** It averages 4 days (on promise) but ranges 2–6 days.
3. **Review low-margin categories** such as Men's Footwear.

## AI feature 1: Ask your data in English
A manager types a question. Gemini (`gemini-flash-latest`) writes SQL, DuckDB runs it on the real data, and the answer comes from the data, not from the AI.
- **Safety:** only SELECT queries run. Queries that change data are blocked.
- **Evaluation (10 questions with known answers):**
  - v1: 9/10 correct. The one miss used `Order Status` instead of `Delivery Status` for "cancelled". The data has two cancellation concepts.
  - v2 (after adding a data-dictionary rule): 9/9 answered correctly; 1 question not answered because of the free-tier rate limit.
  - v2 used a rule learned from the test set, so it is not a fully independent test.

## AI feature 2: Predict late deliveries before shipping
- **Features:** only information known before shipping (shipping mode, market, region, category, department, customer segment, payment type, quantity, discount rate, price, order month and weekday).
- **Excluded (leakage):** actual shipping days, delivery status, shipping date, order status.
- **Time-based split:** trained on orders before 2017-04-22, tested on later orders.

| Method | Accuracy | F1 | ROC-AUC |
|---|---|---|---|
| Always guess "late" | 0.573 | – | – |
| Baseline (shipping-mode rule) | 0.696 | 0.670 | 0.744 |
| Gradient boosting (12 features) | 0.695 | 0.674 | 0.744 |

**Conclusion:** the model does not beat the one-rule baseline. The other features add no signal beyond shipping mode. Recommendation: use the simple rule and fix shipping promises, rather than deploy a complex model.

## Data quality and limitations
- **Signs of generated data:** First Class always takes exactly 2 days; each discount level has almost exactly the same number of items (10,028–10,029).
- **Cancelled orders** (7,754) are marked "not late" in the data, so they were excluded from all lateness analysis.
- **Removed columns:** 9 empty or private (emails, passwords, names, empty descriptions) and 6 exact duplicates.
- **Currency** is not stated in the dataset.
- **The late-2017 volume drop** cannot be explained from this data.

## Project structure
| File | Content |
|---|---|
| `01_explore_data.ipynb` | Data loading, quality checks, cleaning |
| `02_sql_analysis.ipynb` | 6 business questions in SQL |
| `03_excel_summary.xlsx` | PivotTables |
| `04_dashboard.pbix` | Power BI dashboard |
| `05_ai_ask_data.ipynb` | AI question-answering tool + evaluation |
| `06_predict_late.ipynb` | Late-delivery prediction vs baseline |

## How to run
1. Download the dataset into a `data/` folder.
2. `pip install pandas duckdb scikit-learn google-genai python-dotenv jupyter`
3. Create a `.env` file with `GEMINI_API_KEY=your_key`
4. Run the notebooks in order (01 → 06).

## Author
Raja Vinay Kumar Koppula · [LinkedIn](https://www.linkedin.com/in/rajavinaykumarkoppula/)