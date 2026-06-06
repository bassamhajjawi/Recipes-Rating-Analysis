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

## Assessment of Missingness

## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis
