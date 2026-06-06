# Do Longer Recipes Actually Taste Better?

A data science project analyzing the relationship between cooking time and recipe ratings.

By Bassam Hajjawi

---

## Introduction
This project uses a dataset of recipes and ratings from [food.com](https://www.food.com/). 
It contains two files: `RAW_recipes.csv` with ~83,000 recipes, and `RAW_interactions.csv` 
with ~730,000 user reviews. After merging and computing an average rating per recipe, the 
final dataset has **83,194 rows**, one per recipe.

**Central question:** Do recipes with longer cooking times receive higher or lower average ratings?

Cooking time is one of the first things people look at when choosing a recipe. Understanding 
whether it's related to ratings could help both cooks and recipe contributors make better decisions.

The relevant columns for this analysis are:
- `minutes`: total cooking time in minutes
- `n_steps`: number of steps in the recipe
- `n_ingredients`: number of ingredients
- `calories`: parsed from the nutrition column
- `submitted`: date the recipe was submitted
- `avg_rating`: average user rating, our response variable

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

1. **Merge** recipes with interactions on `id` & `recipe_id` (left join).
2. **Replace ratings of 0 with NaN.** Food.com uses "0" to indicate a user wrote a review, but did not include a rating. So we must treat a rating of 0 as missing.
3. **Compute `avg_rating`** per recipe and merge it back into `recipes`.
4. **Split `nutrition` column** into seven numeric columns: `calories`, `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fat`, `carbohydrates`.
5. **Convert `submitted` to a `datetime`** so we can later extract year and month.
6. **Drop extreme `minutes` outliers**. Some entries claim cooking times of hundreds of thousands of minutes. We cap at 1440 minutes (24 hours).

| name                                 |   minutes |   n_steps |   n_ingredients |   calories |   avg_rating |
|:-------------------------------------|----------:|----------:|----------------:|-----------:|-------------:|
| 1 brownies in the world    best ever |        40 |        10 |               9 |      138.4 |            4 |
| 1 in canada chocolate chip cookies   |        45 |        12 |              11 |      595.1 |            5 |
| 412 broccoli casserole               |        40 |         6 |               9 |      194.8 |            5 |
| millionaire pound cake               |       120 |         7 |               7 |      878.3 |            5 |
| 2000 meatloaf                        |        90 |        17 |              13 |      267   |            5 |

### Univariate Analysis

The distribution of cooking times has a right-skewed, with most recipes taking between 20–50 minutes. Even though there are some recipes that take longer than 100 minutes, it seems that the majority of food.com recipes are designed for everyday quick cooking.

<iframe
  src="assets/cooking_time_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

All five cooking time groups show similar rating distributions, with medians near 5 and similar spreads. This suggests that cooking time alone may not be a strong predictor of how well a recipe is rated.

<iframe
  src="assets/rating_by_time.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

This pivot table shows mean average rating by cooking time bucket (rows) and number 
of ingredients bucket (columns). Ratings are consistent across all groups (4.57–4.75), 
but recipes with 0–15 minute cooking times tend to rate slightly higher regardless 
of ingredient count.

| minutes_bucket   |   1-5 |   6-10 |   11-15 |   16+ |
|:-----------------|------:|-------:|--------:|------:|
| 0-15             | 4.683 |  4.659 |   4.673 | 4.755 |
| 16-30            | 4.601 |  4.623 |   4.633 | 4.667 |
| 31-60            | 4.633 |  4.597 |   4.609 | 4.63  |
| 61-120           | 4.615 |  4.617 |   4.636 | 4.642 |
| 121+             | 4.594 |  4.567 |   4.598 | 4.656 |

## Assessment of Missingness

### NMAR Analysis

I believe the `description` column is **NMAR** (Not Missing At Random). On food.com, the description is text the contributor writes about their recipe. A contributor would be more likely to skip the description if they believe the recipe is self-explanatory (e.g., "scrambled eggs") or when they're in a hurry and making a short meal. In both cases the description the contributor would have written about is what determines whether the field is left blank. That makes the missingness depend on the unobserved value itself.

To convert this to MAR, we'd need additional data about the contributor's behavior, for example: how many recipes they've posted or how much time they spent on each submission. With that information, the "in a hurry and low investment" labels could be better observed, making the missingness explainable by observed variables rather than the missing values themselves.

### Missingness Dependency

I'll analyze the missingness of `avg_rating` and test whether it depends on `n_steps` and `n_ingredients`.

For the test statistic, I'll use the Kolmogorov-Smirnov (K-S) statistic. The KS statistic is the maximum 
gap between the two empirical CDFs of the column being tested (one for rows where 
`avg_rating` is missing, one where it isn't). This detects differences anywhere in the 
distribution, not just the means.

I'll use a permutation test with 1,000 shuffles to build the null distribution and compute a p-value. I'll use a significance level of α = 0.01.

**Results**

<iframe
  src="assets/missingness_kde.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

- For `n_steps`, the observed K-S statistic is 0.0758 with a p-value of 0.0000. 
We reject the null at α = 0.01, meaning the missingness of `avg_rating` does depend 
on `n_steps`.

- For `n_ingredients`, the observed K-S statistic is 0.0264 with a p-value of 0.0250. 
We fail to reject the null at α = 0.01 meaning there is no strong evidence that the 
missingness of `avg_rating` depends on `n_ingredients`.


## Hypothesis Testing

- **Null Hypothesis:** Recipes with short cooking times (under 30 minutes) and long cooking times (over 30 minutes) have the same average rating.
- **Alternative Hypothesis:** Recipes with short cooking times (under 30 minutes) and long cooking times (over 30 minutes) have different average ratings.

**Test statistic:** Absolute difference in mean `avg_rating` between the two groups. This fits because the alternative hypothesis is two-sided.

**Significance level:** α = 0.05.

**Procedure:** Permutation test with 1,000 shuffles of group labels.

**Results:** The observed absolute difference in means is **0.035**. Across 1,000 permutations, none produced a test statistic as extreme as the observed value, giving a p-value of 0.0.

**Conclusion:** We reject the null hypothesis at the α = 0.05 level. The data are consistent with short and long recipes having different average ratings, though we cannot prove either hypothesis with certainty.


## Framing a Prediction Problem

**Prediction:** Predict the average user rating (`avg_rating`) of a recipe. This is a regression problem (the response is continuous on a 1.0–5.0 scale).

**Response variable:** `avg_rating` because it summarizes the recipe's overall approval in a single number. Predicting it before any ratings exist is useful for users and the platform.

**Evaluation metric:** Root Mean Squared Error (RMSE). We pick RMSE over MAE because larger prediction errors are disproportionately bad in this context, so a recipe predicted to be 4.9 stars that actually scores 3.0 misleads users much more than two recipes each off by 0.5. Squaring penalizes those large misses.

**Time-of-prediction features:** When a recipe is first posted, the only information 
available is what the contributor provided: `minutes`, `n_steps`, `n_ingredients`, 
and `calories`. We cannot use anything that comes from user interactions (like reviews) since those don't exist yet at the time of posting.

## Baseline Model

The baseline model is `LinearRegression` using two features from the original dataset:

- `minutes`: quantitative (continuous, total cooking time)
- `n_steps`: quantitative (number of steps in the recipe)

Both features are quantitative. Both are left untouched because scaling doesn't affect linear regression predictions.

The data is split 80/20 into train/test sets so we can evaluate generalization to unseen data.

**Performance:** Train RMSE = 0.6395 and Test RMSE = 0.6425. The small gap means the 
model isn't overfitting, but the RMSE is approximately equal to the standard deviation of 
`avg_rating` (0.64), so the model is doing the same as predicting the 
mean rating every time. `minutes` and `n_steps` alone don't carry much signal.

**Plans for improvement:** Add `log(minutes)` and `log(calories)` as engineered 
features (both columns are heavily right-skewed, so the log transform helps), and 
switch to `Lasso` regression with hyperparameter tuning on `alpha` via GridSearchCV.

## Final Model

### Final Model

**Algorithm:** Lasso. The baseline was Linear Regression, so any improvement in test RMSE has to come from the engineered features, not from switching to a more flexible algorithm. Lasso also gives us a hyperparameter, alpha, to tune, which Linear Regression doesn't.

**Features (4 raw and 2 engineered):**

Raw features:
- `n_steps`: kept from the baseline.
- `n_ingredients`: slightly different angle than `n_steps`.
- `calories`: high calorie recipes such as desserts, might have good information to predict the rating.
- `submitted_year`: reviewer behavior might have changed over time with trends or other factors.

Engineered features:
1. **`log_minutes` = log(1 + minutes):** cooking times are heavily right-skewed. In a linear model, raw `minutes` would assume that a 30 to 60 minute jump has the same effect on rating as a 100 to 120 jump, even though the first time is doubling and the second is relatively smaller.
2. **`log_calories` = log(1 + calories):** same right-skewed shape, so the same reasoning here.

For a linear model, log-transforming a feature actually changes the predictions 
(not just the coefficients), so these should be improvements.

**Hyperparameters tuned:** `alpha`, Lasso's regularization strength. I searched 
over `{1e-5, 1e-4, 1e-3, 1e-2, 1e-1, 1.0}` using 5-fold GridSearchCV. A small alpha 
behaves like regular linear regression, while a large alpha will shrink coefficients toward 
zero. I tuned it because some of my features are correlated, like `calories` and 
`log_calories`, so Lasso can reduce the redundancy by shrinking one of them. 
Best alpha: 0.0001.

**Results:**

The final model achieved a test RMSE of 0.6419, down from 0.6425 in the baseline. So it had an improvement of ~0.0006.

The improvement comes from two sources:
- **New features:** `n_ingredients`, `calories`, and `submitted_year`.
- **Log transforms:** `log_minutes` and `log_calories` better capture how these 
skewed features relate to ratings in a model.

The gain is small, but ratings cluster heavily near 5, so much of 
the variance is noise that a model can not predict.

## Fairness Analysis

**Groups:**
- **Group X: Short recipes:** recipes with `minutes ≤ 30`.
- **Group Y: Long recipes:** recipes with `minutes > 30`.

These groups are interesting because cooking time is the central variable in this project. If the model predicts well for quick recipes but poorly for elaborate ones (or the other way around), that's a real fairness concern for users on either side.

**Evaluation metric:** RMSE

**Hypotheses:**
- **Null:** The model is fair. Its RMSE for short and long recipes is the same.
- **Alternative:** The model's RMSE for short recipes is different from its RMSE for long recipes.

**Test statistic:** Absolute difference in RMSE between the two groups.

**Significance level:** α = 0.05.

**Procedure:** A permutation test that shuffles the group label (short vs. long) 1,000 times and computes the test statistic under each shuffle. We use the `final_model`.

**Conclusion:** The observed absolute RMSE difference between short and long recipes was 
0.0288 (short: 0.6260 and long: 0.6549). After 1,000 permutations, the p-value was 
0.0980, which is above our significance level of 0.05. We fail to reject the null 
hypothesis, meaning there is no statistically significant evidence that the model performs 
differently for short vs. long recipes.



