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

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis
