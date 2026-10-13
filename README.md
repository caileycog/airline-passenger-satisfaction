# Predicting Airline Passenger Satisfaction

**Logistic regression vs gradient boosted trees in KNIME: which model predicts satisfied passengers better, and what drives satisfaction?**

![KNIME](https://img.shields.io/badge/KNIME-5.3.3-FDD800?logo=knime&logoColor=black)
![Task](https://img.shields.io/badge/task-binary%20classification-blue)
![Models](https://img.shields.io/badge/models-Logistic%20Regression%20%7C%20Gradient%20Boosted%20Trees-orange)
![Validation](https://img.shields.io/badge/validation-5--fold%20CV%20%2B%20held--out%20test-green)

MSc Data Science, University of Sheffield (2026) · Module: Data Mining and AI · Author: **Cailey Coghlan**

📄 [Read the full report (PDF)](docs/report.pdf)

---

## Project summary

### The problem
Airlines are still recovering from the financial losses of the COVID-19 pandemic, and passenger satisfaction is one of their main sources of competitive advantage. If an airline can predict which passengers will be satisfied, and understand why, it can direct service improvements and loyalty spending where they will have the most effect.

### What I did
I built a complete machine-learning workflow in **KNIME** on **5,196 passenger survey records** with **21 predictors**: demographics, travel details, twelve service ratings, flight distance and delays. The target was whether a passenger was *satisfied* or *neutral/dissatisfied*.

- **Cleaned the data.** I found **580 impossible ages of 999** and **539 service ratings of 0** that meant "not applicable" rather than a poor score, and recoded both as missing. I also assessed the likely cause of missingness for each affected variable (MCAR, MAR or MNAR) before choosing how to treat it.
- **Prevented data leakage.** I split the data **80/20 (stratified)** *before* any preprocessing. Imputation and z-score standardisation were fitted on the training set only and then applied to the test set.
- **Built two models with different strengths.** **Logistic regression** is the interpretable benchmark: it needs imputation, standardisation, chosen reference categories and the removal of one of two highly correlated delay variables (ρ = 0.75). **Gradient boosted trees** handle missing values, mixed scales and correlated predictors natively, so they needed none of that.
- **Tuned and validated both.** I used **5-fold cross-validation** with systematic hyperparameter tuning, then made a single final evaluation on the untouched test set using four metrics: accuracy, F1, AUC-ROC and Cohen's kappa.

### What I found
- **Gradient boosted trees outperformed logistic regression on every metric:** test accuracy **0.90** vs 0.84, F1 **0.89** vs 0.82, AUC-ROC **0.97** vs 0.91, Cohen's kappa **0.80** vs 0.68.
- On the 1,040 test passengers, GBT made **28 fewer false negatives** (64 vs 92) and **36 fewer false positives** (36 vs 72).
- Performance dropped only slightly from cross-validation to the test set, so **neither model overfitted**.
- The logistic regression coefficients identified the **main drivers of satisfaction**:
  - **business travel**: 10.4× the odds of satisfaction compared with personal travel
  - **loyalty cardholders**: 4.5× the odds of non-cardholders
  - **online check-in**: 2.7× the odds of those who didn't use it
  - **business class**: 2.1× the odds of economy
  - **ease of online booking**: +79% odds per rating point
  - **arrival delay**: −30% odds per standard deviation of delay

### Why it matters
These results translate into concrete recommendations: grow the loyalty programme, invest in digital touchpoints (online check-in and booking), improve economy-class comfort, and reduce arrival delays. These are the most controllable levers for raising satisfaction. The analysis also raises a **fairness** point: the patterns favour business and premium passengers, and the data includes protected characteristics (gender, age). So any deployed model should be audited before it influences how passengers are treated.

---

## Key results

| Model | Set | Accuracy | F1 | AUC-ROC | Cohen's κ |
|---|---|---|---|---|---|
| Logistic Regression | Training (5-fold CV) | 0.86 | 0.83 | 0.92 | 0.71 |
| Logistic Regression | Test | 0.84 | 0.82 | 0.91 | 0.68 |
| Gradient Boosted Trees | Training (5-fold CV) | 0.92 | 0.91 | 0.97 | 0.84 |
| **Gradient Boosted Trees** | **Test** | **0.90** | **0.89** | **0.97** | **0.80** |

*F1, precision and recall are for the "satisfied" class. Full-precision values, including precision, recall and specificity, are in [`tables/model_performance.csv`](tables/model_performance.csv).*

<table>
  <tr>
    <td><img src="figures/models/roc_gbt_test.jpg" width="100%"></td>
    <td><img src="figures/models/roc_lr_test.jpg" width="100%"></td>
  </tr>
</table>

### Test-set confusion matrices (n = 1,040)

| | **GBT:** Pred. dissatisfied | **GBT:** Pred. satisfied | **LR:** Pred. dissatisfied | **LR:** Pred. satisfied |
|---|---|---|---|---|
| **Actual dissatisfied** | 550 | 36 | 514 | 72 |
| **Actual satisfied** | 64 | 390 | 92 | 362 |

### Strongest predictors (logistic regression, p < .001)

| Rank | Predictor | β | Odds ratio | Interpretation |
|---|---|---|---|---|
| 1 | Type of travel: business | +2.339 | 10.37 | 937% higher odds than personal travel |
| 2 | Customer loyalty: cardholder | +1.508 | 4.52 | 352% higher odds than non-cardholders |
| 3 | Online check-in: used | +0.977 | 2.66 | 166% higher odds than not using it |
| 4 | Class: business | +0.733 | 2.08 | 108% higher odds than economy |
| 5 | Ease of online booking | +0.580 | 1.79 | +79% odds per rating point |
| 6 | Arrival delay | −0.362 | 0.70 | −30% odds per standard deviation |

*All 21 coefficients, with standard errors, p-values and reference categories, are in [`tables/lr_coefficients.csv`](tables/lr_coefficients.csv).*

---

## The KNIME workflow

<p align="center">
  <img src="figures/workflow_overview.png" width="100%" alt="KNIME workflow">
</p>

The workflow reads the data, cleans it (top left), explores it (statistics, box plots, bar charts, Spearman correlation), and then splits 80/20. From there it branches into two pipelines: **logistic regression** (top), with encoding, imputation, normalisation, a 5-fold CV loop and test scoring, and **gradient boosted trees** (bottom), with a 5-fold CV loop and test scoring.

### Design decisions that matter

- **Split first, preprocess second.** Imputation values and normalisation parameters are learned from the training set only and applied to the test set with KNIME's *(Apply)* nodes, so no test information leaks into training.
- **Each model gets the preprocessing it needs, and only that.** GBT uses the raw cleaned data. Logistic regression gets median imputation for continuous variables and mode imputation for categorical ones, z-score scaling, and manual encoding with reference categories chosen for interpretability.
- **Multicollinearity was handled where it matters.** Departure and arrival delay are strongly correlated (Spearman ρ = 0.75). I removed departure delay from the LR pipeline only, keeping arrival delay because it has far fewer missing values (15 vs 747). GBT kept both.
- **Z-score rather than min-max scaling,** because flight distance and arrival delay contain extreme but valid values that min-max scaling would compress.
- **Metrics chosen for a mildly imbalanced target** (56.3% / 43.7%). Accuracy is reported alongside F1, AUC-ROC and Cohen's kappa, which corrects for chance agreement.
- **Hyperparameter tuning was systematic.**
  - **Logistic regression:** I tested regularisation types and variances. Only Gaussian (L2) with variance 0.1 and 500 epochs converged.
  - **GBT:** I tuned learning rate, then depth, then number of trees, one at a time. The best settings were learning rate 0.1, 100 trees and max depth 6.
  - Full grids are in [`tables/`](tables/).

---

## Exploratory data analysis

<table>
  <tr>
    <td><img src="figures/eda/satisfaction_by_travel_type.jpg" width="100%"></td>
    <td><img src="figures/eda/satisfaction_by_class.jpg" width="100%"></td>
  </tr>
  <tr>
    <td><img src="figures/eda/satisfaction_by_loyalty.jpg" width="100%"></td>
    <td><img src="figures/eda/spearman_correlation_heatmap.jpg" width="100%"></td>
  </tr>
</table>

<details>
<summary>More figures (class balance, online check-in, box plots)</summary>
<br>
<table>
  <tr>
    <td><img src="figures/eda/satisfaction_distribution.jpg" width="100%"></td>
    <td><img src="figures/eda/satisfaction_by_online_checkin.jpg" width="100%"></td>
  </tr>
  <tr>
    <td><img src="figures/eda/boxplot_age.png" width="100%"></td>
    <td><img src="figures/eda/boxplot_flight_distance.png" width="100%"></td>
  </tr>
  <tr>
    <td><img src="figures/eda/boxplot_departure_delay.png" width="100%"></td>
    <td><img src="figures/eda/boxplot_arrival_delay.png" width="100%"></td>
  </tr>
  <tr>
    <td><img src="figures/eda/boxplot_departure_arrival_convenience.png" width="100%"></td>
    <td><img src="figures/eda/boxplot_ease_of_online_booking.png" width="100%"></td>
  </tr>
  <tr>
    <td><img src="figures/eda/boxplot_food_and_drink.png" width="100%"></td>
    <td><img src="figures/eda/boxplot_leg_room_service.png" width="100%"></td>
  </tr>
</table>
</details>

The age box plot shows the **999 values** that were recoded as missing. The four service-rating box plots show the **0 = "not applicable"** responses.

---

## Limitations and next steps

- **Small dataset.** At 5,196 records, it's much smaller than comparable studies (25,000 to 100,000+), which likely explains part of the gap to published accuracies. The dataset also has no in-flight Wi-Fi rating, which is one of the strongest predictors in other studies.
- **Imputation may bias LR coefficients.** Single median and mode imputation could distort the logistic regression estimates. Multiple imputation would preserve relationships better.
- **Categorical multicollinearity wasn't tested.** Testing it with Cramér's V would make the LR coefficients more trustworthy. Gate location's unexpected negative coefficient is a likely symptom.
- **No feature importance for GBT.** KNIME 5.3.3 doesn't display feature importance for gradient boosted trees, so the drivers come from LR only.
- **Final models come from the last CV fold.** Both test-set models are the ones trained in the final cross-validation fold, on about 80% of the training data, rather than refitted on the full training set. This doesn't leak information, but refitting is standard practice and is a planned improvement.
- **Next steps:**
  - try XGBoost or LightGBM on a larger dataset
  - use multiple imputation for LR
  - compute SHAP or permutation importance for GBT, for example through KNIME's Python integration
  - run a fairness audit across gender and age groups

---

## Repository structure

```
airline-passenger-satisfaction/
├── README.md                     # this page
├── workflow/
│   └── Airline-Passenger-Satisfaction.knwf   # the full KNIME workflow
├── data/
│   └── AirlineSatisfactionData.csv           # 5,196 passengers × 23 columns
├── figures/
│   ├── workflow_overview.png     # the KNIME workflow canvas
│   ├── eda/                      # bar charts, box plots, correlation heatmap
│   └── models/                   # ROC curves
├── tables/
│   ├── model_performance.csv     # all metrics, both models, CV and test
│   ├── confusion_matrices_test.csv
│   ├── lr_coefficients.csv       # all 21 coefficients with odds ratios
│   ├── hyperparameter_tuning_lr.csv
│   ├── hyperparameter_tuning_gbt.csv
│   └── knime_outputs/            # original KNIME result screenshots
└── docs/
    ├── report.pdf                # the full report
    └── FILE_GUIDE.md             # what every file is
```

**Guide to every file:** [docs/FILE_GUIDE.md](docs/FILE_GUIDE.md)

---

## How to run

1. Install the free **[KNIME Analytics Platform](https://www.knime.com/downloads)**. The workflow was built in version 5.3.3.
2. Download this repository with **Code → Download ZIP**, then unzip it.
3. In KNIME, choose **File → Import KNIME Workflow…** and select `workflow/Airline-Passenger-Satisfaction.knwf`.
4. Open the workflow and click **Execute all**. The dataset is bundled inside the workflow, so no file paths need changing.
5. To see a result, right-click any Scorer, ROC Curve or chart node and open its view.

Random seeds are fixed in the Partitioning, cross-validation and learner nodes, so results reproduce exactly.

---

## Tech stack

**Platform:** KNIME Analytics Platform 5.3.3 · **Models:** Logistic Regression (SAG optimiser, L2 regularisation), Gradient Boosted Trees · **Validation:** stratified 80/20 split, 5-fold cross-validation (X-Partitioner / X-Aggregator) · **Preprocessing:** Rule Engine, Missing Value, Normalizer, Column Filter, Numeric Outliers · **Evaluation:** Scorer, ROC Curve · **EDA:** Statistics, Box Plot, Bar Chart, Rank Correlation (Spearman)

**Skills shown:** data cleaning and data-quality detection · missing-data mechanisms (MCAR, MAR, MNAR) · leakage-safe preprocessing · feature encoding and scaling · multicollinearity handling · hyperparameter tuning · cross-validation · model comparison · interpreting odds ratios · turning results into business recommendations · fairness awareness

---

## Data source

The dataset (`AirlineSatisfactionData.csv`, 5,196 records) was provided by the module instructor for the University of Sheffield *Data Mining and AI* module. It contains anonymised survey responses with no personal identifiers.

---

## Contact

**Cailey Coghlan**, MSc Data Science, University of Sheffield
[LinkedIn](https://www.linkedin.com/in/cailey-coghlan-10a8a8173/) · [Email](mailto:caileycog@outlook.com)

See also: [Psoriasis Biomarker Discovery with Interpretable Machine Learning](https://github.com/caileycog/psoriasis-biomarker-discovery)
