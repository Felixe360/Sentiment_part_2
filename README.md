# Sentiment_part_2
# 📊 Project Report: Sentiment Portfolio Optimization

## 1. The Core Philosophy

Let’s start with the big idea. You probably know Ben Graham’s famous quote:

> *“In the short run, the stock market is a voting machine. But in the long run, it is a weighing machine.”*

Most quantitative funds focus purely on the **weighing machine**—fundamentals, balance sheets, and long-term factors.  
This project asks a different question:

**Can we mathematically model the _voting machine_?**

We know that fear, greed, and rumors drive short-term price action. The goal of this project was to build a portfolio that doesn’t ignore this noise, but instead **listens to it** in order to maximize returns.

---

## 2. The Data Pipeline

To ensure this wasn’t just a theoretical exercise, I built a rigorous data pipeline with no cherry-picking.

- **Stock Selection:**  
  50 stocks selected randomly across industries and market capitalizations.
- **Benchmarks:**  
  6 factor ETFs (Value, Momentum, Size, etc.) to model systemic risk.
- **News Data:**  
  One full year of company-specific news articles pulled using the Finnhub API.

---

## 3. How Sentiment Is Calculated

This is the engine of the project. Rather than counting “good” vs. “bad” words, I used a modern NLP approach.

### Step A: The Brain (BERT Model)

A BERT-based Large Language Model fine-tuned for financial text.

- **Input:** Raw text from news headlines or summaries  
- **Processing:** Context-aware language understanding  
- **Output:** Probability scores for three labels:
  - Positive
  - Negative
  - Neutral

---

### Step B: The Scoring Formula

We need a numeric signal that can be used mathematically. For each article:

$$
\text{Article Score} = P(\text{Positive}) - P(\text{Negative})
$$


This produces a score between **-1.0** (purely negative) and **+1.0** (purely positive).

---

### Step C: Rolling Window (Smoothing)

One article is noise. A sequence of articles is a signal.

To capture prevailing market mood, sentiment scores were aggregated over a rolling time window.

**Figure 1:** Distribution of BERT sentiment scores.  
The bell-shaped curve demonstrates that the model captures nuanced sentiment rather than binary outcomes.
<img width="773" height="433" alt="Screenshot 2026-01-11 at 3 50 44 PM" src="https://github.com/user-attachments/assets/10713507-660a-4d84-aa44-4e8dbb53f005" />

---

## 4. The Math: Modeling Expected Return

To construct the portfolio, expected returns were required for each stock.

I began with a standard **factor model**, but the factor ETFs exhibited multicollinearity.  
To resolve this, I applied **Principal Component Analysis (PCA)**.

> PCA acts like noise-canceling headphones for data—transforming correlated signals into clean, independent components.

---

### The Sentiment Twist

The PCA-based expected returns were then adjusted using sentiment:

$$
E(r_{\text{final}}) = E(r_{\text{PCA}}) + \alpha \times \text{Sentiment Score}
$$


Where:
- \( \alpha \) controls the influence of sentiment
- Positive news increases expected return
- Negative news decreases expected return

---

## 5. Findings: Timing Is Everything

The strategy works — **but only with the right timing**.

- **< 10 Days:** Too noisy  
- **> 30 Days:** Signal becomes stale  
- **15–20 Days (Sweet Spot):**  
  Captures post-reaction trends before momentum fades

**Figure 2:** Cumulative performance of the Sentiment Strategy (green) vs. Baseline (black).  
The lower panel shows the alpha spread, highlighting consistent excess returns.

---

## 6. Market Overreaction: A Case Study (ROKU, Q3 2024)

Why does this work? Because markets overreact.

**What happened:**
- Revenue and EPS beat expectations
- Guidance slightly uncertain
- Stock dropped ~13%

This was a panic-driven selloff.

While price collapsed, **news sentiment remained positive** due to strong earnings.  
The model detected a divergence: **Price ↓, Sentiment ↑**

**Figure 3:** ROKU Q3 2024 Case Study  
Price drops sharply (blue line), while sentiment scores remain positive (orange bars), signaling a buying opportunity.
<img width="690" height="365" alt="roku" src="https://github.com/user-attachments/assets/d9879957-1658-41e1-a8bd-37de6f223aad" />

---

## 7. Experiments & Validation

## Experiment 1: Alpha Sensitivity Analysis

### Objective
To determine the optimal influence of sentiment on the portfolio. While the **sentiment window** controls how much historical news is considered, **alpha (α)** determines how strongly that sentiment impacts the expected return model.

### Hypothesis
There exists a “sweet spot” for sentiment influence:

- **Low Alpha (α < 0.5):**  
  The sentiment signal is too weak to generate meaningful excess returns.
- **High Alpha (α > 1.2):**  
  The portfolio becomes overly reactive to news, resulting in excessive turnover and volatility.



### Methodology

- **Universe:**  
  A diverse 5-stock subset:
  - XOM  
  - SLV  
  - TGT  
  - SMR  
  - AAPL

- **Control Variables:**  
  - Sentiment windows fixed at **15 days** and **20 days**, identified as high-performing in prior tests.

- **Test Variable:**  
  - Alpha values tested:  
    `0.3, 0.6, 0.9, 1.2, 1.5`

- **Evaluation Metric:**  
  - **Excess Cumulative Return**  
    (Strategy Return − Baseline Return)



### Results

The experiment produced a **two-panel visualization** comparing excess returns across different alpha levels.

Key findings:
- Higher alpha values increased divergence from the benchmark.
- Alpha values in the range of **0.9 to 1.2** delivered the most **consistent outperformance**.
- Beyond this range, gains were offset by increased volatility and instability.

**Conclusion:**  
An alpha between **0.9 and 1.2** represents the optimal balance between sentiment responsiveness and portfolio stability.

### Visualizing Experiment 1

You can use the following description in the report:

**Impact of Sentiment Weight (Alpha) on Excess Returns**

- **(15-Day Window):**  
  Demonstrates how aggressive sentiment weighting (**α = 1.5**) produces higher return peaks but also sharper drawdowns compared to more conservative weighting (**α = 0.3**).

- **(20-Day Window):**  
  Shows a smoother cumulative return profile, confirming that the 20-day sentiment window reduces noise introduced by higher alpha values.


<img width="1306" height="494" alt="15 day, excess return" src="https://github.com/user-attachments/assets/c396576c-8150-4f93-8f87-80e5e9400b62" />

<img width="1306" height="529" alt="20 day , excess return" src="https://github.com/user-attachments/assets/5b35b4fe-babc-4788-bebb-055be214f4c2" />

---


## Experiment 2: Monte Carlo Robustness Test

### Objective
To verify that the strategy’s performance is not a fluke driven by lucky stock selection. By running the simulation **40 times** on random subsets of stocks, this experiment tests whether the **“Sentiment Edge”** is a universal market effect or merely a quirk of specific tickers.



### Hypothesis
If the *Voting Machine* theory is valid, the sentiment-adjusted portfolio should outperform the baseline **on average** across randomly constructed portfolios, regardless of the specific stocks selected.



### Methodology

- **Simulation:**  
  40 Monte Carlo iterations

- **Stock Selection:**  
  In each iteration, **5 random stocks** are drawn from the S&P 500 universe.

- **Variables Tested:**
  - **Sentiment Window:** 15 days vs. 20 days  
  - **Alpha (Aggressiveness):**  
    `0.3, 0.6, 0.9, 1.2, 1.5`

- **Evaluation Metric:**  
  - **Average Excess Return**  
    (Sentiment Strategy − Baseline Strategy), averaged across all 40 simulations

---

### Sentiment Adjustment Formula

The experiment applied a conviction- and risk-aware sentiment adjustment:

**Adjustment** = AvgSentiment × α × ln(1 + NewsVolume) × (0.20 / Volatility)


This formulation ensures:
- Larger position adjustments when **news conviction is high**
- Smaller bets when **stock volatility is elevated**
- Improved risk control across heterogeneous assets

---

### Results

The Monte Carlo analysis confirms that the strategy is **robust**.

- **Consistency:**  
  The **15-day sentiment window with α = 1.2** consistently produced the highest average excess return across random portfolios.

- **Risk Control:**  
  The volatility scaling term \((0.20 / \sigma)\) effectively prevented excessive exposure to unstable stocks, stabilizing performance across simulations.

---

### Visualizing Experiment 2

The code output generates the following two-panel visualization:

**Figure 6: Monte Carlo Simulation Results (40 Iterations)**

- **Top Panel (15-Day Window):**  
  Displays average excess return across alpha levels.  
  The curve for **α = 1.2** typically sits highest, indicating optimal aggressiveness.

- **Bottom Panel (20-Day Window):**  
  Shows similar trends with slightly dampened returns, suggesting the **15-day sentiment signal is sharper and more responsive**.


<img width="1217" height="464" alt="EXP2,  15ave" src="https://github.com/user-attachments/assets/4c13e8ed-6dbd-4840-a58a-7384bf33409c" />

<img width="1217" height="477" alt="exp2, 20 ave" src="https://github.com/user-attachments/assets/133541b0-dc4c-4b54-82bc-c9b29a5c2864" />


  
