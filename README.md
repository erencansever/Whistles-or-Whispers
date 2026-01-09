# Heavy-Bet Study — Football Odds vs Results  
**Selected Major Leagues · Selected Seasons · Calibration & Line‑Movement Analysis**


## Project Overview
This project examines how pre‑match betting expectations relate to actual match outcomes in selected major European football leagues. We focus on whether teams that receive strong market support (proxied by opening→closing line movement) meet, exceed, or underperform their final implied probabilities.

Because actual betting volume is not public, we use line movement as a practical proxy for the “heavily‑backed side.” Data comes from Football‑Data.co.uk (results + opening odds) and multi‑bookmaker closing odds (BTB). Coverage includes England (E0–E3), Germany (D1), France (F1), and Italy (I1) for the seasons available under `raw/Football-Data/`.

The central research question is: **Do teams that receive strong betting market support before kick‑off underperform relative to their final implied probabilities?**

We address this with a reproducible pipeline:
- collect and merge match results with opening and closing odds,
- convert odds to (vig‑free) implied probabilities,
- quantify line movement and identify heavily‑backed sides,
- evaluate calibration and test hypotheses comparing expected vs. realized outcomes.

We do not claim wrongdoing or intentional underperformance. The goal is a transparent, data‑driven assessment of how well betting markets anticipate results and whether heavily‑backed teams deviate meaningfully from expectations.

## Motivation

Betting markets synthesize information quickly, but their relationship to realized outcomes is not perfect. Exploring opening vs. closing odds, how bettors react to new information, and whether those signals align with on‑field results can shed light on market efficiency, calibration, and biases. A concise, data‑driven analysis helps move the discussion from speculation to measurable evidence and improves transparency around what odds do—and don’t—tell us about match outcomes.

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

- Goal: predict `HeavySide_Won` for heavy favorites.
- Data: merges `processed/average_opening_odds_group1.csv` and `processed/average_closing_odds_group1.csv` on `(HomeTeam, AwayTeam, Date, league)`; rows with missing odds are dropped.
- Heavy favorite filter: favorite opening odds < 1.40 (configurable).
- Label: `HeavySide_Won = 1` if the opening favorite (H/A) matches the final result (`FTR`), else 0.
- Features:
	- `pFav_open`, `pFav_close`, `delta_pFav = pFav_close − pFav_open`,
	- `overround_open`, `overround_close`,
	- league dummies derived from `league` (drop-first encoding).
- Preprocessing: implied probs computed from odds with overround normalization; optional row sampling (`sample_size`) for fast runs.
- Train/test: stratified 75/25 split (`random_state=42`).
- Model: Logistic Regression (L2, `liblinear`, `class_weight='balanced'`).
- Metrics: AUC, Brier score, Accuracy, positive class rate, and learned coefficients (saved to `reports/baseline_upset_model_metrics.txt`).
- Config via env vars: `OPENING_CSV`, `CLOSING_CSV`, `HEAVY_FAV_THRESHOLD`, `SAMPLE_SIZE`, `SEED`.


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
