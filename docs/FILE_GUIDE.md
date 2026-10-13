# File Guide

This page explains every file in the repository and how to run the workflow. For the project overview, see the [README](../README.md).

---

## 1. How to run the workflow

1. **Install KNIME Analytics Platform** (free) from [knime.com/downloads](https://www.knime.com/downloads). The workflow was built in **version 5.3.3**. Later 5.x versions should open it too.
2. **Download the repository.** Use the green **Code → Download ZIP** button on GitHub, then unzip it. Alternatively, run:
   ```bash
   git clone https://github.com/caileycog/airline-passenger-satisfaction.git
   ```
3. **Import the workflow.** In KNIME, choose **File → Import KNIME Workflow…**, select `workflow/Airline-Passenger-Satisfaction.knwf` and click **Finish**.
4. **Run it.** Open the workflow and click **Execute all**, the double green arrow in the toolbar. The dataset is stored inside the workflow, so it runs without changing any file paths.
5. **View the results.** Right-click a node and choose its view. For example, the **Scorer** nodes show accuracy, F1 and Cohen's kappa, and the **ROC Curve** nodes show AUC-ROC.

**Troubleshooting**

| Problem | Fix |
|---|---|
| CSV Reader shows an error saying the file can't be found | Open the CSV Reader node and point it at `data/AirlineSatisfactionData.csv` from this repository. |
| KNIME asks to install missing extensions | Accept. The nodes used are all part of standard KNIME. |
| Results differ slightly from the report | Check that you haven't changed the random seeds in the Partitioning, X-Partitioner or learner nodes. |

---

## 2. Workflow (`workflow/`)

| File | Description |
|---|---|
| `Airline-Passenger-Satisfaction.knwf` | The complete KNIME workflow: data cleaning, exploratory analysis, the logistic regression and gradient boosted trees pipelines, cross-validation, and test-set evaluation. |

### What the workflow does, in order

| Stage | Main KNIME nodes | Report section |
|---|---|---|
| Read the data and drop the ID column | CSV Reader, Column Filter | 4.1 |
| Descriptive statistics and exploratory charts | Statistics, Box Plot, GroupBy, Pivot, Bar Chart | 4.1–4.2 |
| Spearman correlation | Rank Correlation | 4.2.5 |
| Recode "0 = not applicable" ratings and Age = 999 as missing | Rule Engine (×4), Numeric Outliers | 4.2 |
| Stratified 80/20 train/test split (seeded) | Partitioning | 4.3.1 |
| **Logistic regression pipeline:** drop departure delay; fit imputation on the training set and apply it to the test set; encode categories; fit z-score normalisation on the training set and apply it to the test set | Column Filter, Missing Value (+ Apply), Rule Engine, Normalizer (+ Apply) | 4.3.2–4.3.3 |
| Logistic regression 5-fold CV and test scoring | X-Partitioner, Logistic Regression Learner/Predictor, X-Aggregator, Scorer, ROC Curve | 5, 6 |
| Gradient boosted trees 5-fold CV and test scoring | X-Partitioner, Gradient Boosted Trees Learner/Predictor, X-Aggregator, Scorer, ROC Curve | 5, 6 |

---

## 3. Data (`data/`)

| File | Description |
|---|---|
| `AirlineSatisfactionData.csv` | 5,196 passenger survey records × 23 columns: an `id`, 21 predictors and the target `satisfaction` (`satisfied` / `neutral or dissatisfied`). Provided by the module instructor. |

**Columns:**
- **Demographics and travel:** Gender, Customer Loyalty, Age, Type of Travel, Class, Online check-in (0/1), Flight Distance
- **Service ratings:** Departure/Arrival time convenient, Ease of Online booking, Gate location, Food and drink, Seat comfort, Inflight entertainment, On-board service, Leg room service, Baggage handling, Checkin service, Inflight service, Cleanliness. These are rated 1–5, and on four of them 0 means "not applicable".
- **Delays:** Departure Delay in Minutes, Arrival Delay in Minutes

**Known data-quality issues, all handled in the workflow:**
- 580 ages of 999
- 539 "not applicable" zeros across four service ratings
- missing values in Online check-in (1,531), Flight Distance (747), Departure Delay (747) and Arrival Delay (15)

---

## 4. Figures (`figures/`)

| File | What it shows |
|---|---|
| `workflow_overview.png` | The full KNIME workflow canvas |
| `eda/satisfaction_distribution.jpg` | Class balance: 2,927 neutral or dissatisfied, 2,269 satisfied |
| `eda/satisfaction_by_travel_type.jpg` | Satisfaction by business or personal travel |
| `eda/satisfaction_by_loyalty.jpg` | Satisfaction by loyalty cardholder status |
| `eda/satisfaction_by_class.jpg` | Satisfaction by travel class |
| `eda/satisfaction_by_online_checkin.jpg` | Satisfaction by online check-in use (0 = no, 1 = yes) |
| `eda/boxplot_age.png` | Age, showing the impossible 999 values |
| `eda/boxplot_flight_distance.png`, `boxplot_departure_delay.png`, `boxplot_arrival_delay.png` | Continuous variables and their outliers |
| `eda/boxplot_departure_arrival_convenience.png`, `boxplot_ease_of_online_booking.png`, `boxplot_food_and_drink.png`, `boxplot_leg_room_service.png` | The four service ratings that contain "not applicable" zeros |
| `eda/spearman_correlation_heatmap.jpg` | Spearman correlations, including departure vs arrival delay (ρ = 0.75) |
| `models/roc_gbt_test.jpg` | Gradient boosted trees ROC curve on the test set (AUC = 0.967) |
| `models/roc_lr_test.jpg` | Logistic regression ROC curve on the test set (AUC = 0.907) |

---

## 5. Tables (`tables/`)

The CSV files were transcribed from the KNIME node outputs, so the results can be read without running KNIME. The original screenshots of those outputs are in `knime_outputs/`.

| File | Contents |
|---|---|
| `model_performance.csv` | Accuracy, F1, AUC-ROC, Cohen's kappa, precision, recall and specificity for both models, under cross-validation and on the test set. Cross-validation AUC is given to two decimal places. |
| `confusion_matrices_test.csv` | Test-set confusion matrices for both models |
| `lr_coefficients.csv` | All logistic regression coefficients, with standard errors, z-scores, p-values, odds ratios and the meaning of each encoded category |
| `hyperparameter_tuning_lr.csv` | Logistic regression configurations tested; only one converged |
| `hyperparameter_tuning_gbt.csv` | Gradient boosted trees configurations tested, with the selected one marked |
| `knime_outputs/lr_cv_scorer.png`, `lr_test_scorer.png`, `lr_test_confusion_matrix.png`, `lr_coefficients.png` | KNIME output: logistic regression |
| `knime_outputs/gbt_cv_scorer.png`, `gbt_test_scorer.png`, `gbt_test_confusion_matrix.png` | KNIME output: gradient boosted trees |
| `knime_outputs/descriptive_statistics_1.png` to `_3.png` | KNIME Statistics node output for the numeric columns |

**How the categories were encoded for logistic regression:**
- Gender: 1 = Female (reference), 2 = Male
- Customer Loyalty: 1 = non-cardholder (reference), 2 = cardholder
- Type of Travel: 1 = Personal (reference), 2 = Business
- Class: 1 = Economy (reference), 2 = Economy Plus, 3 = Business
- Online check-in: 0 = No (reference), 1 = Yes

---

## 6. Documentation (`docs/`)

| File | Description |
|---|---|
| `report.pdf` | The full report: literature, methods, data preparation, tuning, results, discussion and references |
| `FILE_GUIDE.md` | This file |
