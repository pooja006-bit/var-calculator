# Value at Risk (VaR) Calculator

A Python project that answers one simple question every investor should ask:

> **"How much money could I lose on a really bad day?"**

This tool takes a real stock portfolio, downloads 4 years of actual market data, and calculates the answer three different ways then stress tests it against the COVID crash to see how well those answers hold up when things actually go wrong.

---

## What is VaR, in plain English?

VaR (Value at Risk) gives you a number like this:

> *"There's a 95% chance I won't lose more than $2,378 in a single day."*

That remaining 5%? Those are your bad days. VaR tells you where the line is. CVaR (Conditional VaR) tells you what happens *after* you cross the average of your worst days.

---

## The Portfolio

| Stock | What it is | Weight |
|-------|------------|--------|
| AAPL  | Apple | 30% |
| MSFT  | Microsoft | 30% |
| JPM   | JPMorgan Chase | 20% |
| SPY   | S&P 500 ETF (the whole market) | 20% |

**Period:** January 2020 → December 2023 (1,005 trading days)
**Assumed portfolio size:** $100,000

This period was chosen deliberately which includes the COVID crash, the 2021 bull run, and the 2022 rate hike selloff. It's not cherry-picked to look good. 

---

## Three Ways to Calculate VaR

### 1. Historical VaR
Take all 1,005 daily returns, sort them from worst to best, and look at the 5th percentile. No math assumptions but just what actually happened.

### 2. Parametric VaR
Assume returns follow a bell curve (normal distribution), use the portfolio's average return and volatility, and calculate VaR using a formula. Faster, but relies on an assumption that turns out to be wrong (more on that below).

### 3. Monte Carlo VaR
Simulate 10,000 possible future trading days using the portfolio's statistics and find the VaR from those simulations. Sits close to Parametric since it uses the same mean and standard deviation as inputs.

---

## Results

### Core VaR Numbers (95% confidence, $100k portfolio)

| Method | VaR | CVaR | Dollar VaR |
|--------|-----|------|------------|
| Historical | -2.38% | -3.88% | $2,378 |
| Parametric | -2.70% | -3.40% | $2,697 |
| Monte Carlo | ~-2.70% | ~-3.40% | ~$2,700 |

**Reading this:** On a normal bad day (1-in-20), your $100k portfolio would lose around $2,378–$2,697 depending on the method. If you're already in that bad 5%, the average loss there is $3,400–$3,883.

### Backtesting — Did the model actually work?

We checked: over 1,005 days, how many times did real losses actually exceed the VaR threshold?

| Model | Expected Breaches | Actual Breaches | Verdict |
|-------|-------------------|-----------------|---------|
| Historical VaR | 50 days | 51 days (5.07%) | Well-calibrated |
| Parametric VaR | 50 days | 39 days (3.88%) | Slightly off |

Historical VaR nailed it almost exactly. Parametric slightly underestimated the number of bad days because real returns aren't a perfect bell curve.

---

## The Most Important Finding — Returns Aren't Normal

The parametric method assumes stock returns look like a perfect bell curve. They don't.

- **Skewness: -0.097** — slightly more extreme down days than up days
- **Excess Kurtosis: 11.82** — fat tails. Extreme days happen much more often than a normal distribution would predict. A kurtosis of 0 would mean normal. Ours is nearly 12.

This is why at 99% confidence, Historical VaR (-4.42%) is worse than Parametric (-3.85%). The further you go into the tail, the more the bell curve assumption breaks down. Banks use 99% VaR for regulatory reporting and this is exactly where the math gets dangerous if you trust the wrong model.

---

## Sensitivity Analysis — What if you changed the weights?

We tested 6 different ways to split the portfolio:

| Allocation | Daily Volatility | Historical VaR | Dollar VaR |
|------------|-----------------|----------------|------------|
| SPY Heavy (60% SPY) | 1.54% | -2.17% | $2,171 |
| Equal Weights | 1.67% | -2.36% | $2,358 |
| **Current (our portfolio)** | **1.69%** | **-2.38%** | **$2,378** |
| Tech Heavy (90% AAPL+MSFT) | 1.79% | -2.43% | $2,425 |
| Finance Heavy (60% JPM) | 1.80% | -2.54% | $2,544 |
| AAPL Only | 2.12% | -3.24% | $3,241 |

**What this tells you:** Holding only AAPL is 49% riskier than holding mostly SPY. The current portfolio sits right in the middle with decent diversification without being too conservative. Volatility and VaR move almost perfectly in a straight line together, which is exactly what theory predicts.

---

## EWMA VaR — Giving More Weight to Recent Days

Standard VaR treats every day equally. A day from January 2020 counts the same as yesterday. EWMA (Exponentially Weighted Moving Average) uses a decay factor (λ = 0.94, the JP Morgan RiskMetrics standard) so recent days matter more and old data gradually fades.

| | Value |
|--|-------|
| EWMA Sigma | 0.59% |
| Regular Sigma | 1.69% |
| EWMA VaR (95%) | -0.77% ($770) |
| Regular Historical VaR | -2.38% ($2,378) |

EWMA VaR was dramatically lower (-0.77% vs -2.38%) because by end of 2023, markets had been calm for over a year. EWMA had mostly forgotten COVID. This is the tradeoff: EWMA reacts faster to changing conditions, but it also forgets crises faster. In calm times it gives you a lower (more optimistic) VaR. When volatility spikes, it catches up quickly.

---

## Stress Test — What if COVID Happened to Your Portfolio?

This is where everything gets real. We took the actual COVID crash period (Feb 20 – Mar 23, 2020) and measured what it would have done to this portfolio.

| Metric | Value |
|--------|-------|
| Duration | 23 trading days |
| Cumulative Loss | **-37.59%** |
| Dollar Loss | **$37,591** |
| Worst Single Day | -13.49% (March 16, 2020) |
| Crisis Volatility | 5.98% per day |
| Normal Volatility | 1.69% per day |
| Volatility Spike | **3.5x above normal** |
| Days VaR was breached | **13 out of 23 (56.5%)** |
| Expected breach rate | 5% |

VaR said losses beyond -2.38% should only happen 5% of the time. During COVID, it happened 56.5% of the time. The model didn't fail because it was coded wrong but it failed because VaR is designed for normal markets, and COVID was not a normal market. This is the most important lesson in risk management: **VaR tells you nothing about what happens during a crisis. That's exactly why stress testing exists.**

---

## Top 10 Worst Days (real dates, real losses)

| Date | Loss | Dollar Loss |
|------|------|-------------|
| 2020-03-16 | -13.49% | $13,485 |
| 2020-03-09 | -9.35% | $9,354 |
| 2020-03-12 | -9.25% | $9,246 |
| 2020-06-11 | -6.17% | $6,171 |
| 2020-03-18 | -5.75% | $5,747 |
| 2020-02-27 | -5.52% | $5,517 |
| 2020-04-01 | -5.08% | $5,078 |
| 2020-03-27 | -4.80% | $4,796 |
| 2022-09-13 | -4.77% | $4,772 |
| 2020-09-08 | -4.69% | $4,691 |

8 of the 10 worst days were during COVID. The one exception in 2022 was September 13, the day CPI inflation data came in hotter than expected and caused one of the biggest single-day drops of the rate hike era.

---

## Project Structure

```
var-calculator/
│
├── notebooks/
│   ├── 01_var_calculator.ipynb        # Core VaR: Historical, Parametric, Monte Carlo
│   └── 02_sensitivity_analysis.ipynb  # Confidence levels, weights, EWMA, stress test
│
├── data/                              # Generated plots and saved price data
├── src/                               # Helper scripts (future use)
├── requirements.txt
└── README.md
```

---

## How to Run It

```bash
git clone https://github.com/pooja006-bit/var-calculator.git
cd var-calculator
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
jupyter notebook
```

Open `notebooks/var_calculator.ipynb` and run all cells. Then open `sensitivity_analysis.ipynb` for the advanced analysis.

Data is downloaded automatically via `yfinance`, no manual downloads needed.

---

## Tech Stack

Python 3.12 · Pandas 3.0 · NumPy 2.4 · SciPy 1.17 · yfinance 1.2 · Matplotlib · Jupyter

---

## Challenges Faced

**1. No single VaR method is reliable on its own**
The biggest challenge wasn't calculating VaR, it was figuring out which method to trust and when. Historical VaR is accurate but entirely dependent on what happened in your sample period. If your data happens to be mostly calm years, it'll underestimate risk. Parametric VaR is consistent but assumes a bell curve that real markets don't follow. Monte Carlo sits close to Parametric because it uses the same mean and standard deviation as inputs so it doesn't really add new information unless you model the distribution differently. Understanding the tradeoffs between all three, rather than just picking one, was the core challenge of the project.

**2. The normality assumption fails exactly where it matters most**
Parametric VaR gave a tighter (less scary) number than Historical VaR at 99% confidence (-3.85% vs -4.42%). That gap exists because the normal distribution underestimates how often extreme days happen and the further into the tail you go, the worse that underestimation gets. The challenge was understanding why this happens mathematically (excess kurtosis of 11.82 nearly 12x what a normal distribution would have) and what it means practically: the model that's easiest to explain to management is also the one most likely to understate risk during a real crisis.

**3. EWMA gives a completely different picture depending on when you run it**
EWMA VaR came out at -0.77% while Historical VaR was -2.38% — less than a third of the value. Both are technically correct for the same portfolio. The difference is that EWMA weights recent calm periods heavily and lets COVID fade into the background. This creates a real dilemma: EWMA is more responsive to current market conditions, which sounds like a good thing, but it also means it gives you a false sense of safety after a long quiet period right before the next crisis hits. Deciding when EWMA is useful versus misleading required thinking through the context, not just the math.

**4. VaR backtesting revealed a model accuracy gap**
Historical VaR breached on 51 days out of 1,005 (5.07% — almost exactly the expected 5%). Parametric VaR only breached on 39 days (3.88% — meaningfully below the 5% target). On the surface Parametric looks conservative, but what it's actually doing is failing to flag enough bad days. In a risk management context, a model that misses 11 bad days it should have caught is not being conservative it's being inaccurate. Interpreting backtesting results correctly, rather than just reporting the numbers, required understanding what the breach rate is supposed to tell you.

**5. Stress testing showed VaR is the wrong tool for crisis measurement**
The stress test was the hardest result to interpret. VaR said the daily loss threshold would be breached 5% of the time. During the COVID crash it was breached 56.5% of the time on 13 out of 23 days. The challenge here wasn't the calculation, it was understanding that this isn't a failure of the calculation it's a fundamental limitation of what VaR measures. VaR is calibrated on normal market behavior. A crisis, by definition, is not normal market behavior. Explaining why a model that "broke" during COVID is still a useful tool (just not for crisis scenarios) is a genuinely difficult concept to communicate clearly.

---

## What Could Be Done Next

**Use a fat-tailed distribution instead of normal**
The excess kurtosis of 11.82 proves the normal distribution is a bad fit for these returns. A Student's t-distribution or a Cornish-Fisher expansion would model the fat tails much better and make Parametric VaR more accurate especially at 99% confidence where the gap between Historical and Parametric was largest (-4.42% vs -3.85%).

**Add more crisis periods to the stress test**
The project only stress tests against COVID. But the 2008 financial crisis, the 2000 dot-com crash, and the 2022 rate hike year are all worth testing. Each one affected tech, financials, and the broad market differently running all three would show which crisis would have been worst for this specific portfolio.

**Build a proper Monte Carlo with correlation**
The current Monte Carlo simulates each stock independently using just the portfolio mean and sigma. A better version would use the full covariance matrix between AAPL, MSFT, JPM, and SPY so when tech crashes, the simulation knows AAPL and MSFT tend to crash together. That's how real risk systems work.

**Make it interactive**
Right now you have to go into the code and change tickers and weights manually. A simple web app using Streamlit would let anyone type in their own stocks, adjust weights with sliders, and see their VaR update in real time without touching any code.

**Intraday VaR**
This project uses daily returns. But traders who hold positions for hours, not days, need VaR on a 1-hour or 15-minute timeframe. Extending to intraday data (available from some free APIs) would make the tool relevant to a completely different use case.

**Automatic alerts**
If the rolling 30-day VaR crosses a threshold say, it doubles from its recent average that's a signal that volatility is picking up. Adding an automated alert (email or Slack notification) when this happens would turn this from an analysis tool into an actual monitoring system.

---

## What I Learned

VaR is a starting point, not a safety net. It works well in normal markets and breaks completely in crises exactly when you need it most. The Historical method outperformed Parametric in backtesting (51 vs 39 actual breach days vs 50 expected) because it doesn't assume returns are normally distributed. They aren't. An excess kurtosis of 11.82 means extreme days happen far more often than any bell curve would predict. And no model, i.e; Historical, Parametric, or Monte Carlo would have told you that a third of your portfolio was about to disappear in 23 days. That's what stress testing is for.