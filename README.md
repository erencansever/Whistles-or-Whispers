# Whistles or Whispers - Betting Market Anomaly Study 
**Selected Major Leagues · Selected Seasons · Calibration & Line‑Movement Analysis**


## Project Overview

Football betting markets are among the largest and most liquid markets in the world, with billions of dollars wagered every day across leagues, bookmakers, and exchanges. Acording to 2025 estimates, only in Turkey, approximately 60 million USD is wagered daily through legal bookmakers, supported by a base of over 15 million active bettors. These figures alone highlight the extraordinary scale, liquidity, and influence of betting markets to people. 

 With this kind of money attraction, betting markets attracts to many suspissions and speculations to itself too. So i wanted to research one of these suspissions; when big clubs like Real Madrid plays with much much smaller clubs, obviosuly Real Madrid is the favorite and nearly every betmaker plays for them and most of the time wins.  The key question is: " Are those heavy favorites loses the games susspiciously ? "


# Data Collection & Preparation 
-----------------------------------------------
## 1) Data Cleaning

- Standardize team names across Football‑Data and BTB.
- Parse dates to a consistent `datetime` (handle timezone/locale).
- Keep major European leagues (E0–E3, F1, D1, I1) and selected seasons under `raw/Football-Data/`.
- Drop rows with missing/invalid odds; remove duplicates and zero/negative odds.

## 2) Aggregation & Integration

- Base: Football‑Data (results + opening odds).
- Add: BTB multi‑bookmaker closing odds.
- Join key: `(date, home_team, away_team)`.
- Aggregate closing odds across bookmakers (mean of valid values).

## 3) Feature Engineering

- Vig‑free implied probabilities from opening and closing odds.
- Line movement per H/D/A: `ΔOdds`, `ΔProbability`.
- Heavy side = outcome with largest positive `ΔProbability`; target `HeavySide_Won ∈ {0,1}`.
- Extras: `IsFavorite`, `FavoriteProbabilityGap`, `OutcomeCorrectness`.
- Intermediates saved in `processed/` (e.g., averaged opening/closing odds CSVs).

## 4) Reproducibility & Models

- Notebooks: run `baseline_upset_model.ipynb` after data prep.
- Script: `src/baseline_upset_model.py`; metrics in `reports/baseline_upset_model_metrics.txt`.
- League/season filters are parameterizable.


# Analysis Plan

## 1) EDA - (Exploratory Data Analysis)

- Coverage & quality: row counts by league/season, missingness (odds/result), duplicate checks.
- Opening vs. closing odds: distributions per outcome (H/D/A), log‑odds view, league/season comparisons.
- Inter‑bookmaker dispersion: std/iqr across books (opening vs. closing) and outlier detection.
- Line movement: distributions of ΔOdds/ΔProbability; relation to favorite status and upsets.
- Calibration snapshot: implied probability bins vs. realized frequencies (opening vs. closing).
- Heavy‑side diagnostics: win rate by ΔProbability buckets; league/season slices for stability.

## 2) Hypothesis Testing

Hypothesis:
H₀ (Null): Heavy favourites' match outcomes follow expected patterns based on market probabilities. (Y_i ~ Bernoulli(p_i), where p_i = pFav_win)
H₁ (Alternative): Heavy favourites perform significantly different than market expectations

Method:
**Monte Carlo Simulation** - Simulated 20.000 iterations for heavy favorites with significance level of α = 0.05
Results: Failed to reject H₀ 

Method:
**Goodnes of Fit Test** - Measured deviation between observed and expected frequencies with significance level of α = 0.05
Results: Failed to reject H₀ 

Hypothesis - (Closing Odds):
H₀ (Null): The empirical distribution of favorite wins follows the theoretical distribution based on market closing probabilities
H₁ (Alternative): The empirical distribution significantly differs from the theoretical distribution

Method:
**Kolmogorov-Smirnov Test** - Two-sample Kolmogorov-Smirnov test applied to compare empirical vs theoretical distributions with significance level of α = 0.05
Result: Failed to reject H₀ 

# Machine Learning Model 

- **Goal:** predict `HeavySide_Won` for heavy favorites.
- **Data:** merges `processed/average_opening_odds_group1.csv` and `processed/average_closing_odds_group1.csv` on `(HomeTeam, AwayTeam, Date, league)`; rows with missing odds are dropped.
- **Heavy favorite filter:** favorite opening odds < 1.40 (configurable).
- **Label:** `HeavySide_Won = 1` if the opening favorite (H/A) matches the final result (`FTR`), else 0.
- **Features:**
	- `pFav_open`, `pFav_close`, `delta_pFav = pFav_close − pFav_open`,
	- `overround_open`, `overround_close`,
	- league dummies derived from `league` (drop-first encoding).
- **Preprocessing:** implied probs computed from odds with overround normalization; optional row sampling (`sample_size`) for fast runs.
- **Train/test:** stratified 75/25 split (`random_state=42`).
- **Model:** Logistic Regression (L2, `liblinear`, `class_weight='balanced'`).
- **Metrics:** AUC, Brier score, Accuracy, positive class rate, and learned coefficients (saved to `reports/baseline_upset_model_metrics.txt`).
- **Config via env vars:** `OPENING_CSV`, `CLOSING_CSV`, `HEAVY_FAV_THRESHOLD`, `SAMPLE_SIZE`, `SEED`.


# Limitations and Future Work

## Limitations
- Scope limited to major European leagues/seasons in this repo; may not generalize.
- Odds averaging can hide bookmaker differences; possible stale/outlier quotes.
- Heavy‑favorite filter (<1.40) narrows coverage; draw excluded in favorite selection.
- Odds‑only features, no time‑based split, and no PnL/backtest yet.

## Future Work
- Time‑aware splits and expanded leagues/seasons.
- Rich features: team strength (Elo/SPI), form, injuries, rest/weather/referee.
- Book‑level modeling with robust aggregation; calibrated/tree‑based models.
- Strategy backtests with costs/liquidity; report ROI/drawdown.




# Personal Info

## Eren Batu Cansever - ID:32365
