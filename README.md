# Market Intelligence for Used-Car Export

> Data Management Project — Università degli Studi di Milano-Bicocca  
> MSc Data Science — Academic Year 2025/26

---

## Overview

This project builds a **data integration framework** connecting European used-car supply with U.S. consumer preference signals, enabling preference-driven selection of EU listings for potential export. Data from **AutoScout24** (EU supply) and **Cars.com** (U.S. ratings) are scraped, normalized, integrated into a unified dataset, and stored in SQLite. A **Gradient Boosting** classifier (ROC-AUC = 0.88) is trained to predict U.S. consumer appreciation, with LIME-based interpretability analysis revealing the key drivers.

---

## Research Questions

1. **Market matching** — Which EU used-car models are also present in the U.S. market and can be benchmarked against U.S. consumer preferences?
2. **Preference-driven selection** — Which technical characteristics and equipment options are associated with higher U.S. consumer appreciation?
3. **Decision support** — How can the integrated dataset rank EU listings by expected U.S. market appeal?

---

## Pipeline

The project is organized as a notebook-driven pipeline with intermediate artifacts persisted at each stage:

```
AutoScout_Data_Collection.ipynb       → autoscout24_listings.json
                                        autoscout24_car_details.json
                                        autoscout24_combined_data.json

Reviews_Data_Collection.ipynb         → carscom_enriched.json
                                        carscom_flat_weighted.json

Merging_Collections.ipynb             → combined_listings_reviews.csv
                                        my_project.db (SQLite)
                                        combined_listings_reviews_final.csv
```

Logs and checkpoints are stored in `logs/` and `checkpoints/` to ensure traceability and resume capability for long-running collection tasks.

---

## Data Sources

### AutoScout24 — EU Supply
Scraped via `requests` + `BeautifulSoup`, adopting a polite scraping strategy: rotating User-Agent headers, randomized delays between requests, automatic checkpoints every 10 pages, and full exception handling. Snapshot collected on **2025-12-29**.

| Metric | Value |
|--------|-------|
| Raw listings retrieved | 3,633 |
| Duplicates removed | 366 |
| Listings with parsed details | 3,532 |

### Cars.com — U.S. Consumer Preferences
Scraped via `Selenium` with explicit politeness delays. Multi-dimensional ratings extracted at make/model level: `comfort`, `interior`, `performance`, `value`, `exterior`, `reliability`, and review volume (`cumulata`).

---

## Integration & Storage

Integration is performed by normalizing `make` and `model` into slug form and joining EU listings with U.S. ratings on the `(make_slug, model_slug)` key. Only EU vehicles with available U.S. review data are retained.

The integrated dataset is persisted in **SQLite** (`my_project.db`, table `cars_data`) for reproducible querying.

| Metric | Value |
|--------|-------|
| Integrated rows | 1,507 |
| Unique make/model pairs | 239 |
| Columns | 174 |
| Completeness | 98.85% |
| Uniqueness | 99.8% |

---

## Target Definition

The binary target `appreciate` is derived from a **weighted PCA** applied to the six Cars.com rating dimensions, using review volume (`cumulata`) as sample weights. The first principal component (PC1) serves as a composite preference score:

```
appreciate = (PC1 > mean(PC1))
```

| Class | Count |
|-------|-------|
| `appreciate = False` | 936 |
| `appreciate = True` | 550 |

---

## Model & Results

A `GradientBoostingClassifier` is trained on the integrated dataset (80/20 stratified split), with hyperparameter tuning via `GridSearchCV` (5-fold CV, scoring: ROC-AUC).

| Metric | Baseline | Tuned |
|--------|----------|-------|
| Accuracy | 0.8020 | 0.8322 |
| Precision | 0.7576 | 0.7830 |
| Recall | 0.6818 | 0.7545 |
| F1 | 0.7177 | 0.7685 |
| **ROC-AUC** | 0.8571 | **0.8838** |

---

## Interpretability (LIME + Native Importance)

Both native Gradient Boosting feature importance and aggregated LIME explanations are computed. LIME additionally provides directional information.

**Key findings:**
- `spec_body_type_Convertible` — strongest positive contribution to appreciation
- `power` and `spec_engine_size` — positive, consistent with a preference for higher-performance vehicles
- `spec_fuel_type_Electric` — strongest negative contribution
- `spec_body_type_Van` and `feat_right_hand_drive` — negative contributions

---

## Limitations

- **Regulatory feasibility not modeled** — EU-to-U.S. export requires FMVSS/EPA compliance; conversion costs are out of scope
- **Single time snapshot** — EU supply collected on one day; temporal dynamics not captured
- **Cross-market matching constraints** — join limited to make/model alignment and review availability on the U.S. side

---

## Authors

**Nicolò Bachiorri · [Davide Francesco Caramia](https://github.com/davidecar02) · Hervé Mottaran**  
Università degli Studi di Milano-Bicocca
