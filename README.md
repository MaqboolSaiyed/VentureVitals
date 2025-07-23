# Startup Health Scoring – ScaleDux Task 1

## Problem Statement
Early-stage investors, incubators and innovation programs often evaluate **hundreds of startups** at once.  While raw metrics such as _market size_ or _burn rate_ are informative, comparing companies across many disparate indicators is tedious and error–prone.  The goal of this project is therefore to **distil multiple key metrics into a single quantitative _Health Score_** that makes it trivial to rank startups and identify those that warrant deeper due-diligence.

## Solution Overview
A single, self-contained Jupyter notebook (`ScaleDux_Task1.ipynb`) automates the complete workflow:

1. **Environment Setup**  – Installs all required Python packages and mounts Google Drive (when run in Colab) so that both the input CSV and the output figures are persisted.
2. **Data Loading & Inspection** – Reads `Startup_Scoring_Dataset.csv`, prints head, summary statistics and checks for missing values.
3. **Data Pre-processing**
   * **Normalisation** – All numerical features are scaled to zero-mean, unit-variance using `sklearn.preprocessing.StandardScaler` so that one feature does not dominate solely because of its magnitude.
   * **Burn-rate Transformation** – Because a high burn rate is _negative_, its normalised value is **multiplied by –1** so that higher (worse) raw burn rates reduce the final health score.
4. **Health Score Calculation** – A **weighted linear blend** of the normalised features:

   ```python
   health_score = (
       0.25 * team_experience           +
       0.20 * market_size_million_usd   +
       0.20 * monthly_active_users      +
       0.15 * funds_raised_inr          +
       0.20 * (- monthly_burn_rate_inr) # inverted sign
   )
   ```

   The weights were chosen heuristically to emphasise proven traction (active users) and total addressable market, while still rewarding solid teams and fundraising success.
5. **Ranking & Export** – Startups are sorted in descending order of `health_score` and the top 10 are displayed.
6. **Visualisation** – Three figures are generated and saved under the `output/` directory:
   * `top_10_health_scores_bar_chart.png` – Bar chart of the ten best-scoring startups.
   * `feature_correlation_heatmap.png` – Correlation matrix between all input features to reveal multicollinearity.
   * `health_score_distribution_histogram.png` – Distribution of the final health scores.

## Tech Stack
* **Python 3 & Jupyter/Colab** – Interactive environment for code and narrative.
* **Pandas & NumPy** – Fast, idiomatic tabular data manipulation.
* **scikit-learn** – Provides robust, well-tested scaling utilities that future-proof the method for bigger datasets.
* **Matplotlib & Seaborn** – Industry-standard plotting stack offering both low-level control and attractive statistical defaults.
* **Google Colab** – Eliminates local setup friction; anyone with a Google account can open the notebook, connect to a free GPU/CPU instance and reproduce the results in minutes.

## Repository Structure
```
ScaleDux/
├── ScaleDux_Task1.ipynb            # End-to-end notebook (can be renamed to startup_health_scoring.ipynb)
├── Startup_Scoring_Dataset.csv     # Synthetic dataset of 100 startups × 7 columns
├── output/
│   ├── feature_correlation_heatmap.png
│   ├── health_score_distribution_histogram.png
│   └── top_10_health_scores_bar_chart.png
└── README.md                       # ← you are here
```

## Usage
1. Open the notebook in Google Colab _or_ JupyterLab.
2. If using Colab, ensure the CSV is in your Drive at `/content/drive/MyDrive/ScaleDux/` (or update the path in the notebook).
3. **Run all cells** – the notebook installs any missing dependencies, computes the scores and writes the figures to `/content/outputs/` (Colab) or `output/` (local).

## Future Improvements
* Parameterise feature weights and expose a simple UI slider to let analysts adjust the model interactively.
* Replace the heuristic weights with a data-driven approach (e.g. principal component analysis or supervised learning if labelled outcomes become available).
* Add unit tests to validate each processing step and continuous-integration hooks to guarantee reproducibility.

---

## Assignment Q & A

### 1. What were the assignment questions?
1. Design a Colab-ready notebook that ingests `Startup_Scoring_Dataset.csv`.
2. Pre-process the data and derive a single **Health Score** for every startup.
3. Rank the companies, save insightful charts, and explain the underlying rationale.

### 2. Why these variables and how do they matter in real life?
| Variable | Rationale |
|----------|-----------|
| **team_experience** | Teams with a proven track record pivot faster and de-risk execution. |
| **market_size_million_usd** | A larger Total Addressable Market (TAM) caps upside; investors chase outsized potential. |
| **monthly_active_users** | Traction is the strongest proof of product-market fit. |
| **funds_raised_inr** | Capital raised is a proxy for external validation and runway. |
| **monthly_burn_rate_inr** | High burn shortens runway; unsustainable spend is punished (hence the inversion). |
| **valuation_inr** | Gives context to fund-raising efficiency and existing dilution. |

### 2b. Why these specific weights?
The weights (0.20–0.25 for the _positive_ drivers and −0.25 for burn) were chosen by
1. **Industry intuition** – Traction, market size and team quality usually top investors’ check-lists, so they all carry similar weight.
2. **Range balance** – Each feature was scaled 0-1. Equalish weights avoid any single metric dominating after normalisation.
3. **Stress-testing** – Several weight grids were tried in a scratchpad; the chosen set produced rankings that best matched common-sense expectations (e.g. high-user, low-burn companies surfaced near the top).

### 2c. Handling negatively correlated metrics (e.g. burn rate)
After scaling, the burn-rate dimension was **multiplied by −1** before entering the weighted sum. This simple sign flip converts "lower is better" metrics into a positive contribution to the score, while keeping interpretability (higher composite score = healthier).

### 3. Which ML model is used and why?
A transparent **weighted additive linear model** (essentially a hand-crafted linear regression without fitting) was chosen over complex ML models because:
* Stakeholders (founders & investors) need to _see_ how each factor moves the score.
* The dataset is small (100 rows) and synthetic, so expressive models would over-fit.
* Weight tweaks can be decided by domain experts without re-training code.

### 4. Why invert the monthly burn rate?
Because _lower_ burn is _better_. Multiplying its scaled value by **−1** ensures the health score **decreases** for startups that consume cash faster, accurately penalising financial indiscipline.

### 5. What do the visualisation charts say?
* **Top-10 Bar Chart** – Reveals the clear front-runners and the margin between them.
* **Correlation Heatmap** – Shows relationships among features and confirms burn rate’s negative influence.
* **Score Distribution Histogram** – Illustrates how scores spread across the cohort; most firms fall mid-range.

---
**Author:** Maqbool Husain Saiyed  

*Last updated: 23 Jul 2025*
