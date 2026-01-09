# Heavy-Bet Anomaly Study — Football Odds vs Results  
**Six Major Leagues · Three Seasons · Calibration/Anomaly Analysis**

---

## Contents

- Project Overview  
- Motivation  
- Data Sources  
- Data Collection and Preparation
- Research Questions and Hypothesis
- Methodology

---

## Project Overview
This project explores the relationship between betting market expectations and real match outcomes in 5 major European football leagues to look for an answer to the widespread speculations about rigged referees, match fairness, and potential suspicious matches, the project aims to investigate how "stronger" teams perform relative to the bookmaker odds.

Using multi season, real world football datasets, we try to examine whether teams that experience significant support in the betting markets (measured through opening and closing odds movement) tend to meet, exceed, or underperform their expected probabilities. Since actual bet volume is not publicly available, line movement serves as a practical proxy for identifying the “heavily-backed side.”

The central research question is: **"Do teams that receive strong betting market support before kick-off underperform relative to their final implied probabilities?"**

To answer this, the project conducts an end-to-end analysis pipeline:
- Collecting match results from the five major European leagues across three consecutive seasons,  
- Merging them with bookmaker opening and closing odds,  
- Converting odds into implied probabilities,  
- Identifying matches with significant line movement,  
- Performing calibration tests and anomaly detection to compare expected vs. actual outcomes.

The aim is not to confirm wrongdoing or intentional underperformance, but rather to provide a systematic, transparent, and reproducible statistical assessment. By grounding the discussion in data—rather than anecdotes or sensational claims—the project seeks to offer clearer insight into how well betting markets anticipate real match results and whether heavily-backed teams deviate meaningfully from market expectations.

---

## Motivation

The relationship between betting market expectations and actual football match outcomes has become an increasingly relevant topic in sports analytics. As a data science enthusiast with a strong interest in football-related behavioral patterns, I am motivated to explore this issue through rigorous, data-driven methods rather than public speculation. Understanding how odds evolve, how bettors collectively react to information, and whether these reactions align with real performance can provide valuable insights into market efficiency, forecasting accuracy, and potential biases. Such analysis not only contributes to a more objective understanding of match expectations, but also helps inform broader discussions about transparency, fairness, and the reliability of market-based predictions in modern football.

---

## Data Sources

### 1. Football-Data.co.uk Dataset (Kaggle Mirror)

**Source:** https://www.kaggle.com/datasets/jamieandrews/footballdatacouk  
**Original Website:** https://www.football-data.co.uk/

**Contains:**

- Match results (goals, outcomes)  
- Dates and league information  
- Pre-match **opening odds** from multiple bookmakers  

**Usage in this project:**

- Serves as one of the two core datasets of this project
- Provides **opening odds** needed to compute initial implied probabilities  
- Used to build the foundation of the merged dataset (team names, leagues, dates, openning odds, closing odds)

This dataset is not clean, not standardized, and covers major European leagues



### 2. Beat-the-Bookie Worldwide Football Dataset

**Source:** https://www.kaggle.com/datasets/austro/beat-the-bookie-worldwide-football-dataset  

**Contains:**

- **Opening and closing odds** from multiple bookmakers  
- Multi-league global football coverage  
- High-resolution pricing data near kickoff  

**Usage in this project:**

- Second core dataset of this project
- Provides **closing odds**, essential for line-movement analysis
- Allows comparison of opening vs closing implied probabilities  
- Enables identification of the **“heavily-backed” side** on **heavily-played** matches
 
This dataset fills the gap that Football-Data.co.uk lacks (closing odds), and again not clean nor standardized.

---
## Data Collection and Preparation

### 1. Data Cleaning

- Standardized team names across both datasets to ensure accurate merging.  
- Converted all date fields to a consistent `datetime` format.  
- Filtered matches to include only the six major European leagues and the selected three seasons.  
- Removed rows with missing or inconsistent odds values (e.g., missing home/draw/away odds).  
- Eliminated impossible or erroneous entries, such as negative odds or matches duplicated across bookmakers.  



### 2. Aggregation & Integration

- **Football-Data.co.uk:** Extracted match results and opening odds as the base dataset.  
- **Beat-the-Bookie:** Extracted multi-bookmaker closing odds to capture line movement.  
- Created a canonical match index using *(date, home team, away team)* as the join key.  
- Merged datasets on this index to produce a unified structure containing:  
  - match results  
  - opening odds  
  - closing odds  
  - league metadata  
- Ensured bookmaker-level odds were aggregated appropriately (e.g., averaging across bookmakers when needed).  



### 3. Feature Engineering

- Computed **vig-free implied probabilities** for both opening and closing odds.  
- Calculated **line movement** features:  
  - ΔOdds (closing − opening)  
  - ΔProbability (closing_prob − opening_prob)  
- Identified the **heavily-backed side** as the outcome with the largest positive probability shift.  
- Created binary classification targets:  
  - `HeavySide_Won` (1 if heavily-backed side wins, else 0)  
- Generated additional match-level features such as  
  - `IsFavorite`  
  - `FavoriteProbabilityGap`  
  - `OutcomeCorrectness` (whether the market-predicted favorite actually won)  
- Normalized numeric columns for machine learning models and encoded categorical variables (teams, leagues).  



### 4. EDA

---
