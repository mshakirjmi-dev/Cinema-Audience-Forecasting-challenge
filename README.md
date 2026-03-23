# 🎬 Cinema Audience Forecasting Challenge

> **MLP Project T32025** — Predict daily theatre audience counts across multiple locations.

[![Competition](https://img.shields.io/badge/Competition-Cinema%20Audience%20Forecasting-blue)](https://www.kaggle.com/)
[![Status](https://img.shields.io/badge/Status-Late%20Submission-orange)](https://www.kaggle.com/)
[![Python](https://img.shields.io/badge/Python-3.11-green)](https://python.org)
[![LightGBM](https://img.shields.io/badge/Model-LightGBM-brightgreen)](https://lightgbm.readthedocs.io/)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Competition Details](#-competition-details)
- [Dataset Description](#-dataset-description)
- [Methodology](#-methodology)
- [Results](#-results)
- [Feature Engineering](#-feature-engineering)
- [Model Architecture](#-model-architecture)
- [Installation & Setup](#-installation--setup)
- [Usage](#-usage)
- [Submission Format](#-submission-format)

---

## 🎯 Overview

This project tackles the **Cinema Audience Forecasting** challenge — predicting daily audience counts across multiple theater locations. The goal is to build an accurate ML model that forecasts how many people will attend a cinema on a given day, enabling theaters to optimize staffing, resource allocation, and inventory management.

**Competition Timeline:**
- Start: Sep 15, 2025
- Close: Nov 30, 2025

---

## 🏆 Competition Details

| Item | Details |
|------|---------|
| **Competition** | Cinema Audience Forecasting |
| **Type** | Regression (time-series) |
| **Metric** | Mean Absolute Error (MAE) |
| **Dataset Size** | ~214K training rows |
| **Test Size** | ~38K rows |
| **Best Validation MAE** | ~14.09 |
| **Best Validation R²** | ~0.559 |

### Submission Instructions

1. Join the competition on Kaggle and go to the **Code** tab
2. Create a New Notebook named: `YourRollNo-notebook-t32025` *(e.g., `21f1001234-notebook-t32025`)*
3. Share the notebook with `iitmbscs2008p` collaborator (View access, keep private)
4. Set your team name to your **Kaggle user ID**
5. Submit notebook details via the Competition Registration Form

---

## 📊 Dataset Description

The dataset contains multi-source cinema data. All files are located in `/kaggle/input/Cinema_Audience_Forecasting_challenge/`.

| File | Shape | Description |
|------|-------|-------------|
| `booknow_visits/booknow_visits.csv` | (214046, 3) | Daily audience counts per theater *(target variable)* |
| `booknow_booking/booknow_booking.csv` | (68336, 4) | Online booking records with timestamps |
| `cinePOS_booking/cinePOS_booking.csv` | (1641966, 4) | Point-of-sale ticket transactions |
| `booknow_theaters/booknow_theaters.csv` | (829, 5) | Theater metadata (type, area, lat/lon) |
| `cinePOS_theaters/cinePOS_theaters.csv` | (4690, 5) | POS system theater metadata |
| `movie_theater_id_relation/movie_theater_id_relation.csv` | (150, 2) | Mapping between booking & POS theater IDs |
| `date_info/date_info.csv` | (547, 2) | Day-of-week labels for each date |
| `sample_submission/sample_submission.csv` | (38062, 2) | Test set IDs and submission format |

### Target Variable

`audience_count` — Daily number of attendees per theater (from `booknow_visits`)

---

## 🔬 Methodology

### Pipeline Overview

```
Raw Data → Feature Engineering → Time-based Train/Val Split → Model Training → Prediction → Submission
```

### Step-by-Step

1. **Data Loading & Merging**
   - Load all 7 CSV data sources
   - Aggregate bookings (`booknow`, `cinePOS`) to daily granularity per theater
   - Map `cinePOS` theater IDs to `booknow` IDs via the relation table
   - Join audience data with theater metadata and date info

2. **Exploratory Data Analysis**
   - Distribution analysis of audience counts
   - Time-series trend visualization
   - Theater-level performance comparison
   - Correlation analysis (audience vs. tickets booked/sold)

3. **Feature Engineering** *(16 features total)*
   - Temporal features (day-of-week, weekend flag, month)
   - Lag features (1, 3, 7, 14 days)
   - Rolling mean features (3, 7, 14-day windows)
   - Target-encoded theater type and area means
   - Geographic features (latitude, longitude)
   - Booking signal features (tickets booked, tickets sold)

4. **Train/Validation Split**
   - Time-based split: last 45 days → validation, rest → training
   - No data leakage from future to past

5. **Model Training & Comparison**
   - Ridge Regression (baseline)
   - Random Forest
   - **LightGBM** *(selected — best MAE)*

6. **Final Prediction**
   - Refit best model on full training data (train + validation)
   - Generate test predictions with historical lag features from training history

---

## 📈 Results

### Model Comparison on Validation Set

| Model | MAE | RMSE | R² |
|-------|-----|------|-----|
| Ridge Regression | 15.04 | 22.84 | 0.450 |
| Random Forest | 14.15 | 20.64 | 0.551 |
| **LightGBM** | **14.09** | **20.44** | **0.559** |

**Winner: LightGBM** (lowest MAE on validation set)

### Feature Importance (LightGBM — by gain)

| Rank | Feature | Description |
|------|---------|-------------|
| 1 | `roll_mean_7` | 7-day rolling average audience |
| 2 | `roll_mean_14` | 14-day rolling average audience |
| 3 | `dow` | Day of week |
| 4 | `lag_7` | Audience count 7 days ago |
| 5 | `tickets_booked` | Daily booked tickets |
| 6 | `lag_1` | Audience count yesterday |
| 7 | `lag_14` | Audience count 14 days ago |
| 8 | `roll_mean_3` | 3-day rolling average |
| 9 | `lag_3` | Audience count 3 days ago |
| 10 | `month` | Calendar month |

---

