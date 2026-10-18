# Supply Chain Operations Intelligence

I analysed 180,519 order items to find out why orders arrive late, what causes losses, and what a company should fix first.

**Tools:** Python, SQL (DuckDB), Excel, Power BI, Google Gemini API, scikit-learn

**Results at a glance**
- 57% of shipped orders arrive late, and shipping mode explains most of it.
- Late orders are not less profitable: about 19% of items lose money either way.
- An AI assistant that turns English questions into SQL answered 9 of 10 test questions correctly.

## The problem
A global retailer ships through four shipping modes. More than half of orders arrive late, and about 1 in 5 order items loses money. I wanted to find the causes and the first fixes.

## Data
- [DataCo Smart Supply Chain dataset (Kaggle)](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis)
- 180,519 order items, 53 columns, January 2015 to January 2018
- Public dataset, not a real company (see Limitations)

## What I did
1. Cleaned the data: removed 9 empty or private columns and 6 duplicate columns, checked for missing values and duplicate rows.
2. Answered 6 business questions in SQL.
3. Summarised results in Excel PivotTables.
4. Built a Power BI dashboard with filters for market and shipping mode.
5. Built an AI tool that writes SQL from plain English questions, and tested it.
6. Built a model to predict late deliveries and compared it with a simple rule.

## Findings
1. Shipping mode drives lateness. Late rate: First Class 100%, Second Class 79.8%, Same Day 47.9%, Standard Class 39.8%.
2. The delivery promises don't match reality. First Class promises 1 day but always takes 2. Second Class promises 2 days but takes about 4, the same as Standard Class.
3. Standard Class has the lowest late rate but the most late orders (41.4% of all late items), because it handles the most volume.
4. Region barely matters: late rates range from 54.7% to 60.1%.
5. Late orders don't lose more money: 18.6% of on-time items lose money vs 18.7% of late items.
6. Discounts and categories don't change the loss rate either (about 17–20% everywhere).
7. Profit depends on product mix: Fishing earns 43.6 per item, Men's Footwear 14.0.
8. Lateness stayed between 55% and 59% every month for 3 years.

## Recommendations
1. Make First and Second Class promises match real delivery times, or speed these services up. This covers 54% of late orders. Changing promises sets honest expectations but doesn't make deliveries faster.
2. Make Standard Class more consistent. It averages 4 days but ranges from 2 to 6.
3. Review low-margin categories such as Men's Footwear.

## AI tool: ask questions in English
You type a question, Gemini writes the SQL, and DuckDB runs it on the real data. The answer comes from the data, not from the AI. Only read-only (SELECT) queries are allowed.

I tested it on 10 questions where I already knew the correct answer:
- First run: 9 of 10 correct. The one mistake happened because the data has two different "cancelled" columns, and the AI picked the other one.
- After I added a rule explaining which column to use, it answered that question correctly. (This second run reused the same questions, so it isn't a fully independent test.)

## Predicting late deliveries
I used only information available before an order ships, such as shipping mode, region, product category and order date. I left out anything known only after delivery.

I trained on orders before 22 April 2017 and tested on later orders.

| Method | Accuracy | F1 | ROC-AUC |
|---|---|---|---|
| Always guess "late" | 0.573 | – | – |
| Simple rule (shipping mode only) | 0.696 | 0.670 | 0.744 |
| Gradient boosting (12 features) | 0.695 | 0.674 | 0.744 |

The model did no better than the simple rule. The extra features added nothing beyond shipping mode, so I'd recommend the simple rule plus fixing the delivery promises.

## Limitations
- The data looks partly generated: First Class always takes exactly 2 days, and every discount level has almost the same number of items.
- 7,754 cancelled orders are marked "not late" in the data, so I excluded them from all lateness analysis.
- The currency isn't stated.
- Order volume dropped about 60% from late 2017, and the data doesn't explain why.

## What I learned
- Testing a common belief (late orders lose money) and finding it false was more useful than confirming it.
- A complex model isn't always better. Here a one-line rule worked just as well.
- Unclear data definitions caused the AI's only mistake, so documenting columns matters.

## Files
| File | What it contains |
|---|---|
| `01_explore_data.ipynb` | Data loading, quality checks, cleaning |
| `02_sql_analysis.ipynb` | 6 business questions in SQL |
| `03_excel_summary.xlsx` | PivotTables |
| `04_dashboard.pbix` | Power BI dashboard |
| `05_ai_ask_data.ipynb` | AI SQL assistant and its test |
| `06_predict_late.ipynb` | Late-delivery model vs simple rule |

## How to run
1. Download the dataset into a `data/` folder.
2. `pip install pandas duckdb scikit-learn google-genai python-dotenv jupyter`
3. Create a `.env` file with `GEMINI_API_KEY=your_key`
4. Run the notebooks in order (01 to 06).

## Author
Raja Vinay Kumar Koppula · [LinkedIn](https://www.linkedin.com/in/rajavinaykumarkoppula/)
