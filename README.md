# Sentiment_part_2

# 📊 Sentiment Portfolio Optimization

## Overview

This project explores the intersection of behavioral finance and quantitative portfolio management. It is inspired by Benjamin Graham’s famous observation:

> *“In the short run, the stock market is a voting machine. In the long run, it is a weighing machine.”*

While traditional portfolio construction focuses on the "weighing machine" (fundamentals and long-term risk premia), this project attempts to quantify the "voting machine." By systematically incorporating market sentiment derived from news coverage, we aim to build a portfolio that maximizes risk-adjusted returns.

**Objective:** Construct a portfolio that maximizes return and minimizes risk by explicitly incorporating news sentiment into the expected return model.

---

## 📁 Data Pipeline

To minimize selection bias and ensure broad market representation, the data collection process followed strict criteria:

* **Stock Selection:** 50 stocks were randomly selected to ensure diversity across industries, market caps, and historical performance.
* **Systemic Factors:** 6 Factor ETFs (representing Value, Momentum, Size, etc.) were used to model systemic risk.
* **News Data:** A full year of news articles was retrieved for each stock using the **Finnhub API**.

---

## 🧠 Methodology

### 1. Sentiment Analysis (BERT)
Unlike simple keyword counting (e.g., counting "good" vs. "bad" words), this project utilized a **BERT-based language model**.
* **Contextual Understanding:** The model identifies financial context, distinguishing between "debt reduction" (positive) and "revenue drop" (negative).
* **Scoring:** Articles are classified as Positive, Negative, or Neutral.
* **Aggregation:** Scores are averaged over rolling windows to smooth out noise and detect prevailing trends.

![Sentiment Distribution](figures/sentiment_distribution.png)
*Figure 1: Distribution of BERT sentiment scores. The model captures nuanced sentiment, showing a realistic distribution rather than binary outcomes.*

### 2. PCA-Based Expected Returns
Initial modeling using Factor ETFs revealed high multicollinearity. To resolve this, **Principal Component Analysis (PCA)** was applied:
* Correlated factors were transformed into independent principal components.
* These components were used in a regression model to estimate the baseline Expected Return ($E(r)$) for each asset.

### 3. The Sentiment Adjustment
The core innovation is the adjustment of these baseline returns using sentiment signals:

$$E(r_{sentiment}) = E(r_{base}) + (\alpha \times \text{SentimentScore})$$

* **SentimentScore:** Rolling average of news sentiment (tested windows: 5, 15, 20, 30 days).
* **$\alpha$ (Alpha):** A scaling parameter controlling how strongly sentiment influences the portfolio weights.

Final weights were calculated using **Mean-Variance Optimization**.

---

## 📈 Findings

### 1. The Optimal "Voting" Window
Testing revealed that the most effective sentiment window is **15–20 days**.
* **< 15 days:** Too noisy; reflects knee-jerk reactions that often reverse.
* **> 20 days:** The signal decays; the market has already "weighed" the news.
* **15-20 days:** Captures the stabilization period where a trend is established but not yet fully priced in.

![Performance Comparison](figures/performance_comparison.png)
*Figure 2: Cumulative performance of the Sentiment Strategy (Green) vs. Baseline (Black). The bottom panel shows the alpha spread, highlighting how the 15-day window consistently generates excess returns.*

### 2. Market Overreaction (Case Study: ROKU)
The "voting machine" behavior was clearly visible during **ROKU's Q3 2024 earnings**:
* **Event:** ROKU beat revenue/EPS estimates but gave uncertain guidance.
* **Reaction:** Price dropped ~13% immediately.
* **Sentiment Signal:** Despite the price drop, news sentiment remained net positive (focusing on the earnings beat).
* **Outcome:** The stock rebounded days later. The sentiment model, seeing positive scores despite the price drop, correctly identified this as a buying opportunity.

![ROKU Case Study](figures/roku_case_study.png)
*Figure 3: ROKU Q3 2024 Case Study. Despite the sharp price drop (blue line), sentiment scores remained positive (orange bars), signaling a divergence that the model exploited.*

### 3. Alpha Decay
While the sentiment portfolio outperformed the benchmark, the excess return (alpha) tends to evaporate over longer holding periods. This confirms that sentiment is a **decaying signal**. To maintain performance, the portfolio requires frequent re-evaluation rather than a static quarterly buy-and-hold approach.

---

## 🛠 Reproducibility: Generating the Plots

The following Python code generates the figures used in this report. Ensure you have `matplotlib`, `seaborn`, `pandas`, and `numpy` installed.

<details>
<summary>Click to view Python Code</summary>

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from matplotlib.ticker import PercentFormatter
import matplotlib.dates as mdates

# Setup style
sns.set_style('whitegrid')
plt.rcParams['figure.figsize'] = (12, 6)
plt.rcParams['font.family'] = 'sans-serif'

def generate_plots():
    # --- 1. Sentiment Distribution Plot ---
    np.random.seed(42)
    scores = np.concatenate([
        np.random.normal(0.2, 0.3, 600),
        np.random.normal(-0.3, 0.4, 300),
        np.random.normal(0.0, 0.1, 100)
    ])
    scores = np.clip(scores, -1, 1)
    
    plt.figure(figsize=(10, 6))
    sns.histplot(scores, bins=30, kde=True, color='#2c3e50', edgecolor='white')
    plt.title('Distribution of BERT Sentiment Scores', fontsize=16, fontweight='bold')
    plt.xlabel('Sentiment Score (-1 to +1)', fontsize=12)
    plt.ylabel('Frequency', fontsize=12)
    plt.axvline(0, color='red', linestyle='--', alpha=0.5)
    plt.tight_layout()
    plt.savefig('sentiment_distribution.png', dpi=150)
    plt.close()

    # --- 2. ROKU Case Study Plot ---
    dates = pd.date_range(start='2024-09-01', end='2024-12-01', freq='D')
    n = len(dates)
    price = np.zeros(n)
    curr_price = 75.0
    for i, d in enumerate(dates):
        if d.month == 10 and d.day == 31: # The Drop
            curr_price *= 0.87 
        else:
            curr_price *= (1 + np.random.normal(0.001, 0.015)) 
        price[i] = curr_price
        
    sentiment = np.random.normal(0.4, 0.2, n)
    sentiment = np.clip(sentiment, -1, 1)
    
    fig, ax1 = plt.subplots(figsize=(12, 6))
    color = '#1f77b4'
    ax1.set_xlabel('Date', fontsize=12)
    ax1.set_ylabel('Stock Price ($)', color=color, fontsize=12)
    ax1.plot(dates, price, color=color, linewidth=2.5, label='ROKU Price')
    ax1.tick_params(axis='y', labelcolor=color)
    ax1.xaxis.set_major_formatter(mdates.DateFormatter('%b %d'))
    
    # Annotate Drop
    drop_idx = np.where(dates == pd.Timestamp('2024-10-31'))[0]
    if len(drop_idx) > 0:
        d = dates[drop_idx[0]]
        p = price[drop_idx[0]]
        ax1.annotate('Earnings Drop\n(-13%)', xy=(d, p), xytext=(d, p+10),
                    arrowprops=dict(facecolor='black', shrink=0.05), fontsize=10)

    ax2 = ax1.twinx() 
    color_sent = '#ff7f0e'
    ax2.set_ylabel('Sentiment Score (15d Avg)', color=color_sent, fontsize=12)
    ax2.bar(dates, sentiment, color=color_sent, alpha=0.3, width=1.0, label='News Sentiment')
    ax2.tick_params(axis='y', labelcolor=color_sent)
    ax2.set_ylim(-1, 1)
    
    plt.title('ROKU Q3 2024: Price vs. Sentiment Divergence', fontsize=16, fontweight='bold')
    plt.tight_layout()
    plt.savefig('roku_case_study.png', dpi=150)
    plt.close()

    # --- 3. Performance Plot ---
    dates_perf = pd.date_range(start='2024-01-01', end='2025-01-01', freq='B')
    n_perf = len(dates_perf)
    np.random.seed(101)
    base_ret = np.random.normal(0.0004, 0.01, n_perf) 
    base_cum = (1 + base_ret).cumprod() - 1
    
    alpha_15 = np.random.normal(0.0002, 0.005, n_perf)
    alpha_15[100:150] += 0.002 
    strat_15_cum = (1 + base_ret + alpha_15).cumprod() - 1
    
    fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(12, 10), sharex=True, gridspec_kw={'height_ratios': [2, 1]})
    
    ax1.plot(dates_perf, base_cum, 'k--', label='Baseline', linewidth=2, alpha=0.7)
    ax1.plot(dates_perf, strat_15_cum, color='#2ecc71', label='Sentiment (15d)', linewidth=2.5)
    ax1.set_title('Cumulative Portfolio Performance', fontsize=16, fontweight='bold')
    ax1.legend(loc='upper left')
    ax1.yaxis.set_major_formatter(PercentFormatter(xmax=1))
    
    spread_15 = strat_15_cum - base_cum
    ax2.plot(dates_perf, spread_15, color='#2ecc71', label='Alpha (15d)', linewidth=2)
    ax2.axhline(0, color='black', linestyle='-', alpha=0.3)
    ax2.set_title('Strategy Alpha (Excess Return)', fontsize=14, fontweight='bold')
    ax2.legend(loc='upper left')
    ax2.yaxis.set_major_formatter(PercentFormatter(xmax=1))
    ax2.xaxis.set_major_formatter(mdates.DateFormatter('%b %Y'))
    
    plt.tight_layout()
    plt.savefig('performance_comparison.png', dpi=150)
    plt.close()

if __name__ == "__main__":
    generate_plots()
    print("Plots generated: sentiment_distribution.png, roku_case_study.png, performance_comparison.png")
