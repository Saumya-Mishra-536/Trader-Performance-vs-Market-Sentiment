# Primetrade.ai — Round-0 Assignment
## Trader Performance vs Market Sentiment (Fear/Greed Index)

---

## 📁 Project Structure

```
primetrade_project/
├── analysis_notebook.ipynb      # Jupyter notebook (full reproducible analysis)
├── README.md                    # This file
└── charts/
    ├── 01_pnl_distribution_sentiment.png
    ├── 02_mean_performance_sentiment.png
    ├── 03_behavior_by_sentiment.png
    ├── 04_segment_performance.png
    ├── 05_heatmap_segment_sentiment.png
    ├── 06_long_short_ratio_sentiment.png
    ├── 07_pnl_timeline.png
    ├── 08_trader_archetypes.png
    └── 09_model_results.png
```

---

## ⚙️ Setup & How to Run

### Requirements
```bash
pip install pandas numpy matplotlib seaborn scikit-learn jupyter nbformat
```

### Run the analysis
```bash
jupyter notebook analysis_notebook.ipynb
```

> **Note:** The data files `fear_greed_index.csv` and `historical_data.csv` should be placed in the `data/` directory relative to the notebook.

---

## 📊 Methodology

### Data Preparation (Part A)
- Loaded Fear/Greed Index (2,644 rows, 4 cols) and Hyperliquid trade history (211,224 rows, 16 cols)
- Zero missing values and zero duplicates in both datasets
- Parsed trade timestamps from `Timestamp IST` field (format: `DD-MM-YYYY HH:MM`) and normalized to date
- Merged on daily date key → **211,218 trades across 479 trading days** retained after inner join
- Simplified 5-class Fear/Greed into binary (Fear / Greed) for cleaner analysis; retained original classification for granularity
- Computed daily aggregations per trader: PnL, win rate, trade count, avg size, long/short ratio, fees

### Analysis (Part B)
- **B1:** Compared mean PnL, median PnL, win rate, and % profitable days across Fear vs Greed
- **B2:** Tested behavioral shift in trade frequency, size, and long/short bias by sentiment
- **B3:** Segmented traders by leverage level, trading frequency, and consistency score
- **Bonus:** KMeans clustering (k=4) on 5 trader features to identify behavioral archetypes
- **Bonus:** GradientBoostingClassifier to predict next-day profitability (CV AUC = 0.9715)

---

## 🔍 Key Insights

### Insight 1 — Fear days skew PnL high, but Greed days are more consistently profitable
Fear days produce a higher mean PnL ($5,185 vs $3,973), but this is driven by outlier winners. Greed days have a higher median PnL ($243 vs $123) and a higher share of profitable traders (63.8% vs 60.4%). In practice, Greed days are "safer" for the average trader.

### Insight 2 — Traders overtrade on Fear days
Trade frequency rises ~27% and average trade size rises ~38% on Fear days vs Greed days. This suggests reactive, emotionally-driven trading — not smarter positioning. Higher fees on Fear days further erode net returns.

### Insight 3 — Consistent Winners outperform on fewer days
Consistent Winners trade on ~45 active days vs 98 for Inconsistent traders, yet post higher PnL. Less is more: disciplined, selective traders beat churners even with lower raw activity.

---

## 🎯 Strategy Recommendations (Part C)

### Strategy 1 — "Fear Day Caution Protocol" (for High-Leverage traders)
> **Rule:** On Fear days, cap position size at 75% of normal and avoid new long opens in the first 2 hours. Reserve aggressive leverage for Greed days where directional clarity is higher.
>
> **Rationale:** High-leverage traders see outsized mean PnL on Fear days, but with extreme variance. Protecting against tail losses on Fear days and deploying full leverage on Greed days maximizes risk-adjusted returns.

### Strategy 2 — "Frequency Discipline" (for Frequent traders)
> **Rule:** On Greed days, consolidate into fewer, larger-conviction trades (target <60% of usual trade count). On Fear days, reduce total trades by 30–40% and bias toward short-side entries.
>
> **Rationale:** Frequent traders incur the most fee drag. On Greed days when win-rate peaks, larger but fewer trades capture the same upside with lower friction. On Fear days, reduced frequency limits exposure to choppy, loss-inducing markets.

### Bonus Strategy 3 — Predictive Signal
> **Rule:** Use a rolling 5-day win rate + current sentiment as a binary signal. If rolling win rate < 0.35 AND sentiment == Fear → reduce all exposure by 50%.
>
> **Rationale:** The GBM model achieves 97.15% CV AUC predicting profitable days. The strongest predictor is trailing win rate, which captures momentum in trader skill/market fit.

---

## 📈 Model Results

| Metric | Value |
|--------|-------|
| Model | GradientBoostingClassifier |
| CV AUC (5-fold) | **0.9715 ± 0.0145** |
| Test Accuracy | **93%** |
| Test Precision (Profit) | 0.92 |
| Test Recall (Profit) | 0.98 |

Top features: `win_rate` > `num_trades` > `avg_size_usd` > `fg_value` > `sentiment_binary`

---

*Submitted for Primetrade.ai Data Science Intern — Round 0*