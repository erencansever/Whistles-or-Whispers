<p align="center">
  <img src="https://img.shields.io/badge/Whistles%20or%20Whispers-Betting%20Market%20Study-2d3436?style=for-the-badge&labelColor=0984e3" alt="Project Badge" />
</p>

<h1 align="center">⚽ Whistles or Whispers</h1>
<h3 align="center">Betting Market Anomaly Study</h3>
<p align="center">
  <em>Selected Major Leagues · Selected Seasons · Calibration & Line‑Movement Analysis</em>
</p>

---

## 📋 Table of Contents

- [Overview](#-project-overview)
- [Data Pipeline](#-data-collection--preparation)
- [Analysis](#-analysis-plan)
- [Machine Learning](#-machine-learning-model)
- [Limitations & Roadmap](#-limitations-and-future-work)

---

## 🎯 Project Overview

Football betting markets are among the **largest and most liquid markets** in the world, with billions of dollars wagered every day across leagues, bookmakers, and exchanges. According to 2025 estimates, approximately **60 million USD** is wagered daily through legal bookmakers in Turkey alone, supported by over **15 million active bettors**. These figures highlight the extraordinary scale, liquidity, and influence of betting markets.

With this level of capital flow, betting markets naturally attract suspicions and speculation. This project investigates one such hypothesis:

> **"When big clubs like Real Madrid play much smaller clubs, they are heavy favorites—nearly every bettor backs them and usually wins. But are heavy favorites losing games suspiciously often?"**

---

## 🔧 Data Collection & Preparation

### 1. Data Cleaning

| Task | Description |
|------|-------------|
| **Standardization** | Unify team names across Football‑Data and BTB sources |
| **Date Parsing** | Consistent `datetime` format (timezone/locale handling) |
| **League Filter** | Major European leagues (E0–E3, F1, D1, I1), selected seasons under `raw/Football-Data/` |
| **Quality Control** | Drop rows with missing/invalid odds; remove duplicates and zero/negative odds |

### 2. Aggregation & Integration

- **Base:** Football‑Data (results + opening odds)
- **Supplement:** BTB multi‑bookmaker closing odds
- **Join key:** `(date, home_team, away_team)`
- **Aggregation:** Mean of valid closing odds across bookmakers

### 3. Feature Engineering

| Feature | Description |
|---------|-------------|
| Implied probabilities | Vig‑free, derived from opening and closing odds |
| Line movement | `ΔOdds`, `ΔProbability` per H/D/A |
| Heavy side | Outcome with largest positive `ΔProbability` |
| Target | `HeavySide_Won ∈ {0, 1}` |
| Extras | `IsFavorite`, `FavoriteProbabilityGap`, `OutcomeCorrectness` |

Intermediates saved in `processed/` (e.g., averaged opening/closing odds CSVs).

### 4. Reproducibility & Models

- **Notebooks:** Run `baseline_upset_model.ipynb` after data prep
- **Script:** `src/baseline_upset_model.py`
- **Metrics:** `reports/baseline_upset_model_metrics.txt`
- **Parameters:** League/season filters are configurable

---

## 📊 Analysis Plan

### 1. Exploratory Data Analysis (EDA)

- **Coverage & quality:** Row counts by league/season, missingness, duplicate checks
- **Opening vs. closing odds:** Distributions per H/D/A, log‑odds view, league/season comparisons
- **Inter‑bookmaker dispersion:** Std/IQR across books; outlier detection
- **Line movement:** Distributions of ΔOdds/ΔProbability; relation to favorite status and upsets
- **Calibration snapshot:** Implied probability bins vs. realized frequencies
- **Heavy‑side diagnostics:** Win rate by ΔProbability buckets; league/season slices

### 2. Hypothesis Testing

#### Hypothesis (Opening Odds)

| | Statement |
|-|-----------|
| **H₀ (Null)** | Heavy favourites' match outcomes follow expected patterns based on market probabilities (Yᵢ ~ Bernoulli(pᵢ), where pᵢ = pFav_win) |
| **H₁ (Alternative)** | Heavy favourites perform significantly different than market expectations |

| Method | Result |
|--------|--------|
| **Monte Carlo Simulation** (20,000 iterations, α = 0.05) | Failed to reject H₀ |
| **Goodness of Fit Test** (α = 0.05) | Failed to reject H₀ |

#### Hypothesis (Closing Odds)

| | Statement |
|-|-----------|
| **H₀** | The empirical distribution of favorite wins follows the theoretical distribution based on market closing probabilities |
| **H₁** | The empirical distribution significantly differs from the theoretical distribution |

| Method | Result |
|--------|--------|
| **Kolmogorov–Smirnov Test** (α = 0.05) | Failed to reject H₀ |

---

## 🤖 Machine Learning Model

**Goal:** Predict `HeavySide_Won` for heavy favorites.

| Component | Specification |
|-----------|---------------|
| **Data** | Merge of `processed/average_opening_odds_group1.csv` and `processed/average_closing_odds_group1.csv` on `(HomeTeam, AwayTeam, Date, league)` |
| **Heavy favorite filter** | Opening odds < 1.40 (configurable) |
| **Label** | `HeavySide_Won = 1` if opening favorite matches `FTR`, else 0 |
| **Features** | `pFav_open`, `pFav_close`, `delta_pFav`, `overround_open`, `overround_close`, league dummies |
| **Split** | Stratified 75/25 (random_state=42) |
| **Model** | Logistic Regression (L2, liblinear, class_weight='balanced') |
| **Metrics** | AUC, Brier score, Accuracy, positive class rate, coefficients |

**Config via env vars:** `OPENING_CSV`, `CLOSING_CSV`, `HEAVY_FAV_THRESHOLD`, `SAMPLE_SIZE`, `SEED`.

---

## ⚠️ Limitations and Future Work

### Limitations

- Scope limited to major European leagues/seasons; may not generalize
- Odds averaging can hide bookmaker differences; possible stale/outlier quotes
- Heavy‑favorite filter (<1.40) narrows coverage; draw excluded in favorite selection
- Odds‑only features; no time‑based split; no PnL/backtest yet

### Future Work

- Time‑aware splits and expanded leagues/seasons
- Rich features: team strength (Elo/SPI), form, injuries, rest, weather, referee
- Book‑level modeling with robust aggregation; calibrated/tree‑based models
- Strategy backtests with costs/liquidity; report ROI/drawdown

---

## 👤 Author

**Eren Batu Cansever** · ID: 32365
