<p align="center">
## Whistles or Whisper
  <span style="color: #6c757d;">Betting Market Anomaly Study</span>
</p>

<p align="center">
  <em>Selected Major Leagues · Selected Seasons · Calibration & Line-Movement Analysis</em>
</p>

<p align="center">
  <sub>—</sub>
</p>

---

## Table of Contents

- [Project Overview](#project-overview)
- [Data Collection & Preparation](#data-collection--preparation)
- [Analysis Plan](#analysis-plan)
- [Machine Learning Model](#machine-learning-model)
- [Limitations and Future Work](#limitations-and-future-work)
- [Author](#author)

---

## Project Overview

Football betting markets are among the largest and most liquid markets in the world, with billions of dollars wagered every day across leagues, bookmakers, and exchanges. According to 2025 estimates, approximately **60 million USD** is wagered daily through legal bookmakers in Turkey alone, supported by over **15 million active bettors**. These figures highlight the extraordinary scale, liquidity, and influence of betting markets.

With this level of capital flow, betting markets naturally attract suspicions and speculation. This project investigates one such hypothesis:

> When big clubs like Real Madrid face much smaller teams, they are heavy favorites—nearly every bettor backs them and usually wins. But do heavy favorites lose games **suspiciously often**?

The central research question is: Do favorite teams suffer unexpected losses at an abnormal frequency, beyond what market probabilities would suggest?

---

## Data Collection & Preparation

### 1. Data Cleaning

Raw data is standardized before analysis:

| Task | Description |
|------|-------------|
| **Name standardization** | Team names are unified across Football-Data and BTB sources. |
| **Date parsing** | All dates are parsed into a consistent `datetime` format; timezone and locale differences are resolved. |
| **League filter** | Major European leagues (E0–E3, F1, D1, I1) and selected seasons are kept under `raw/Football-Data/`. |
| **Quality control** | Rows with missing or invalid odds are dropped; duplicates and zero/negative odds are removed. |

### 2. Aggregation & Integration

Two data sources are merged:

- **Primary source:** Football-Data (match results and opening odds).
- **Supplementary source:** BTB multi-bookmaker closing odds.
- **Join key:** `(date, home_team, away_team)`.
- **Aggregation:** Valid closing odds are averaged across bookmakers.

### 3. Feature Engineering

Variables used for modeling and analysis are derived:

| Feature | Description |
|---------|-------------|
| Implied probabilities | Vig-free probabilities computed from opening and closing odds. |
| Line movement | `ΔOdds` and `ΔProbability` are calculated for each outcome (H/D/A). |
| Heavy side | The outcome with the largest positive `ΔProbability` is defined as the heavy side. |
| Target variable | `HeavySide_Won ∈ {0, 1}` — whether the heavy side won the match. |
| Additional variables | `IsFavorite`, `FavoriteProbabilityGap`, `OutcomeCorrectness`. |

Intermediate results are stored in the `processed/` folder (e.g., averaged opening/closing odds CSVs).

### 4. Reproducibility & Models

To ensure reproducibility:

- **Notebook:** Run `baseline_upset_model.ipynb` after data preparation.
- **Script:** `src/baseline_upset_model.py`.
- **Metrics output:** `reports/baseline_upset_model_metrics.txt`.
- **Parameters:** League and season filters are configurable.

---

## Analysis Plan

### 1. Exploratory Data Analysis (EDA)

The structure and quality of the dataset are examined:

- **Coverage and quality:** Row counts by league/season, missing values, duplicate checks.
- **Opening vs. closing odds:** Distributions by outcome (H/D/A), log-odds view, league/season comparisons.
- **Inter-bookmaker dispersion:** Std/IQR across books for opening and closing odds; outlier detection.
- **Line movement:** Distributions of ΔOdds and ΔProbability; relationship to favorite status and upsets.
- **Calibration:** Implied probability bins vs. realized frequencies (opening vs. closing).
- **Heavy-side diagnostics:** Win rate by ΔProbability buckets; stability across league/season slices.

### 2. Hypothesis Testing

#### Opening Odds Hypothesis

| Hypothesis | Statement |
|------------|-----------|
| **H₀ (Null)** | Heavy favourites' match outcomes follow the expected distribution based on market probabilities (Yᵢ ~ Bernoulli(pᵢ), where pᵢ = pFav_win). |
| **H₁ (Alternative)** | Heavy favourites perform significantly differently than market expectations. |

| Method | Result |
|--------|--------|
| **Monte Carlo simulation** (20,000 iterations, α = 0.05) | Failed to reject H₀. |
| **Goodness of fit test** (α = 0.05) | Failed to reject H₀. |

#### Closing Odds Hypothesis

| Hypothesis | Statement |
|------------|-----------|
| **H₀** | The empirical distribution of favorite wins follows the theoretical distribution based on market closing probabilities. |
| **H₁** | The empirical distribution differs significantly from the theoretical distribution. |

| Method | Result |
|--------|--------|
| **Kolmogorov–Smirnov test** (α = 0.05) | Failed to reject H₀. |

In summary, the results indicate that favorite teams’ match outcomes are consistent with market expectations in the current dataset; no abnormal loss frequency was detected.

---

## Machine Learning Model

**Goal:** Predict `HeavySide_Won` for heavy favorites (i.e., whether the favorite wins or an upset occurs).

| Component | Description |
|-----------|-------------|
| **Data** | Merge of `processed/average_opening_odds_group1.csv` and `processed/average_closing_odds_group1.csv` on `(HomeTeam, AwayTeam, Date, league)`. Rows with missing odds are dropped. |
| **Heavy favorite filter** | Matches with opening favorite odds < 1.40 are selected (threshold is configurable via parameter). |
| **Label** | `HeavySide_Won = 1` if the opening favorite (home or away) matches the final match result (`FTR`), else 0. |
| **Features** | `pFav_open`, `pFav_close`, `delta_pFav`, `overround_open`, `overround_close`, league dummies. |
| **Split** | Stratified 75/25 train-test split (`random_state=42`). |
| **Model** | Logistic regression (L2, liblinear, `class_weight='balanced'`). |
| **Metrics** | AUC, Brier score, Accuracy, positive class rate, learned coefficients. |

Configuration via environment variables: `OPENING_CSV`, `CLOSING_CSV`, `HEAVY_FAV_THRESHOLD`, `SAMPLE_SIZE`, `SEED`.

---

## Limitations and Future Work

### Limitations

- Scope is limited to the selected European leagues and seasons in this repository; generalization may be limited.
- Averaging odds across bookmakers can obscure cross-book differences; stale or outlier quotations are possible.
- The heavy favorite filter (<1.40) narrows the sample size; draws are excluded from favorite selection.
- Only odds-based features are used; no time-based split or PnL/backtest is implemented yet.

### Future Work

- Time-aware train-test split and expanded league/season coverage.
- Richer features: team strength (Elo/SPI), form, injuries, rest, weather, referee.
- Book-level modeling with robust aggregation; calibrated or tree-based models.
- Strategy backtests including costs and liquidity; ROI and drawdown reporting.

---

## Author

**Eren Batu Cansever** · ID: 32365
