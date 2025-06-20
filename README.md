# The Secret Sauce to Recipe Ratings: What Makes a Recipe Shine?
Authors: Audrey Chung & Amrutha Potluri

## Overview

This project analyzes recipes from [Food.com](https://www.food.com/?ref=nav) to uncover what factors make up the "secret sauce" to achieve high user ratings. By cleaning the data, engineering features, and building predictive models, we explore which recipe traits drive better reviews.

## Introduction

The **Recipes and Ratings** dataset offers a rich look into user-submitted recipes, including ingredients, preparation steps, nutritional content, and user feedback in the form of ratings. With over 80,000 recipes and nearly 800,000 interactions, it allows us to explore patterns that influence how well a recipe is received.

In this project, we investigate the question:

> **What recipe characteristics are associated with higher user ratings?**

---

This question is important for people with any level of cooking experience. Furthermore, for users, understanding which traits are linked to favorable ratings can guide recipe selection and creation. For content platforms, identifying these features could improve personalization and recommendation systems.

To explore this, we used two CSV files: 
- `RAW_recipes.csv`: Contains metadata for each recipe, such as ingredients, steps, and nutritional values.
- `interactions.csv`: Contains user ratings for each recipe.

`recipe`, contains 83782 rows, with 8 columns that are relevant to our investigation.

| Column Name      | Description                                                                    |
| ---------------- | ------------------------------------------------------------------------------ |
| `name`           | Recipe name                                                                    |
| `id`             | Recipe ID                                                                      |
| `minutes`        | Minutes to prepare recipe                                                      |
| `submitted`      | Date recipe was submitted                                                      |
| `nutrition`      | Nutrition info: \[calories, fat, sugar, sodium, protein, saturated fat, carbs] |
| `n_steps`        | Number of steps in recipe                                                      |
| `n_ingredients`  | Number of ingredients in the recipe                                            |
| `description`    | User-provided description                                                      |


`interactions`, contains 731927 rows, with 3 columns that are relevant to our investigation.

| Column Name | Description                             |
| ----------- | --------------------------------------- |
| `recipe_id` | Recipe ID (connects to `id` in Recipes) |
| `date`      | Date of rating or review                |
| `rating`    | Rating given                            |

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

In order to prepare our data:
- We merged the recipes and interactions datasets.
    - In particular, we performed a left merge on the`recipes` and `interactions` datasets on `id` and `recipe_id`, ensuring that all recipes were retained even if they had no ratings.
- We replaced all ratings of 0 with NaN.
    - Since ratings typically range from 1 to 5, we assume that ratings of 0 indicate a missing rating rather than an extremely low rating. Thus, we replaced 0s with NaN values in order to prevent the 0s from mathematically skewing our calculations which would potentially introduce bias into our results.
- We computed the average rating per recipe, creating the `avg_rating` column.
- We created a binary `high_rating` column if the average rating surpassed an arbitrary threshold.
    - Because the distribution of the average ratings is heavily skewed left, we determined if the average rating for a recipe is **4.5** or greater, it would be considered a high rating. Otherwise, it would be considered a low rating.
- We dropped any recipes that had missing values in the `submitted` column so that we could extract a `day_of_week` column. We then determined if the day of the week fell on a weekend, creating the `is_weekend` column.
    - We wanted to explore whether the timing of recipe submissions correlates with user ratings. Typically, recipes submitted on weekends may reflect more elaborate or carefully prepared dishes, potentially leading to higher ratings. Conversely, weekday recipes may be faster or more casual, which could influence user preferences differently.
- We parsed the nutrition column into separate columns like `sugar_pdv`, `protein_pdv`, and `calories`.
    - We wanted to conclude if health concerns or benefits could affect recipe ratings.
- We created a `desc_len` column which is essentially the number of characters in the description of the recipe.
    - We wanted to test the idea of simplicity, determining if shorter or longer descriptions could have an effect on the recipe's rating.

| **Column Name**  | **Description**                                                           |
|------------------|---------------------------------------------------------------------------|
| `name`           | The name of the recipe                                                    |
| `avg_rating`     | The average user rating                                                   |
| `n_ingredients`  | Number of ingredients in the recipe                                       |
| `n_steps`        | Number of steps in the recipe's instructions                              |
| `minutes`        | Total time (in minutes) to prepare the recipe                             |
| `calories`       | Number of calories in a recipe                                            |
| `total_fat_pdv`  | Percent daily value of total fats in a recipe                             |
| `sugar_pdv`      | Percent daily value of sugar in a recipe                                  |
| `sodium_pdv`     | Percent daily value of sodium in a recipe                                 |
| `protein_pdv`    | Percent daily value of protein in a recipe                                |
| `sat_fat_pdv`    | Percent daily value of saturated fats in a recipe                         |
| `carbs_pdv`      | Percent daily value of carbohydrates in a recipe                          |
| `high_rating`    | Whether the average rating is at least 4.5                                |
| `day_of_week`    | Day of the week the recipe was submitted (derived from `submitted`)       |
| `is_weekend`     | Boolean indicating if it was submitted on a weekend (Sat/Sun)             |
| `desc_len`       | Length of the description (number of characters in the description)       |


Because our DataFrame contains many columns, we only included the most relevant columns in the head of our cleaned DataFrame below:


| name                                 |   minutes |   n_steps |   n_ingredients |   avg_rating | high_rating   | is_weekend   |
|:-------------------------------------|----------:|----------:|----------------:|-------------:|:--------------|:-------------|
| 1 brownies in the world    best ever |        40 |        10 |               9 |            4 | False         | False        |
| 1 in canada chocolate chip cookies   |        45 |        12 |              11 |            5 | True          | False        |
| 412 broccoli casserole               |        40 |         6 |               9 |            5 | True          | False        |
| millionaire pound cake               |       120 |         7 |               7 |            5 | True          | False        |
| 2000 meatloaf                        |        90 |        17 |              13 |            5 | True          | False        |

---

### Univariate Analysis

We analyzed individual variables to understand their distributions and identify patterns that might influence recipe ratings.

#### Average Rating (`avg_rating`)
- Most recipes have high average ratings between 4.5 and 5.0.
- This strong left skew motivated the creation of a binary `high_rating` variable (`avg_rating` ≥ 4.5).

<iframe
  src="assets/univariate1.html"
  width="700"
  height="400"
  frameborder="0"
></iframe>

#### High Rating (`high_rating`)
- A more detailed and closer look at the distribution of higher ratings.

<iframe
  src="assets/univariate2.html"
  width="700"
  height="400"
  frameborder="0"
></iframe>

---

### Bivariate Analysis

We explored relationships between recipe characteristics and average rating to identify potentially predictive features.

#### `n_ingredients` vs. `high_rating`
The graph displays a parabolic shape, indicating that there are higher ratings at the more extreme ends of the spectrum while the graph consistently maintains lower ratings for less extreme values.

<iframe
  src="assets/BIVARIATE1.html"
  width="700"
  height="400"
  frameborder="0"
></iframe>

#### `n_steps` vs. `high_rating`
Generally, the less the number of steps, the lower the dispersion. As the number of steps increases, the variability in the ratings also increases.

<iframe
  src="assets/bivariate3.html"
  width="700"
  height="400"
  frameborder="0"
></iframe>

---

### Interesting Aggregates

#### Average Rating by Number of Steps
Grouped by every 10 `n_steps`, we observed that there is little to no difference in the average ratings. 

| Number of Steps       |    Mean |
|:--------------------- |--------:|
| 0-10                  | 4.62419 |
| 10-20                 | 4.62345 |
| 20-30                 | 4.63594 |
| 30-40                 | 4.68632 |
| 40-50                 | 4.70977 |


This aggregate helped us recognize that other factors may have a stronger role in impacting average ratings. For instance, a user might give 5 stars to both a quick 3-step snack and a fancy 25-step dinner because the different recipes are suited for a different purpose.

---

## Assessment of Missingness

### NMAR Analysis

We examined whether any missing data in our dataset is likely **Not Missing at Random (NMAR)**.

The column, `description`, contains missing values, which conveys that the recipe creator did not submit a description for their recipe. We determined that descriptions would be *NMAR* if their absence is related to qualities we don’t observe in the dataset. For instance, the recipe creator might decide to skip on writing a description if the recipe is very basic or if they're rushing to submit the recipe. 

Because these underlying factors aren’t captured in the dataset, the missingness in `description` is not related to other columns. 
Thus, we believe that `description` is **likely NMAR** where its missingness may be directly related to its own value, not other columns. 

---

### Missingness Dependency: Permutation Tests

To further assess missingness, we performed permutation tests to determine whether the missingness of `avg_rating` depends on other observed features.

#### Does missingness in `avg_rating` depend on `minutes`?

- **Null Hypothesis (H₀)**: Missingness in `avg_rating` is independent of `minutes`.
- **Alternative Hypothesis (H₁)**: Missingness in `avg_rating` depends on `minutes`.

<iframe
  src="assets/miss_min.html"
  width="700"
  height="400"
  frameborder="0"
></iframe>

We computed the observed difference in mean `minutes` between recipes, both with and without a rating. Then, we ran 1000 permutations by shuffling the missingness labels and recomputing the mean difference each time.

<iframe
  src="assets/perm_min.html"
  width="700"
  height="400"
  frameborder="0"
></iframe>

- **Observed Difference**: 117.34
- **p-value**: 0.037

**Conclusion**: Since the p-value is less than 0.05, we reject the null hypothesis. **Missingness in `avg_rating` is not independent of `minutes`.**


---

#### Does missingness in `avg_rating` depend on `day_of_week`?

- **Null Hypothesis (H₀)**: The distribution of missingness in `avg_rating` is the same across all days of the week.
- **Alternative Hypothesis (H₁)**: The missingness in `avg_rating` varies by day of the week.

<iframe
  src="assets/miss_day.html"
  width="700"
  height="400"
  frameborder="0"
></iframe>

We calculated the observed difference between the maximum and minimum missing value rates across days, then compared it to a null distribution generated by permuting the missingness indicator.

<iframe
  src="assets/perm_day.html"
  width="700"
  height="400"
  frameborder="0"
></iframe>

- **Observed Difference**: 0.005
- **p-value**: 0.467

**Conclusion**: Since the p-value is greater than 0.05, we fail to reject the null hypothesis. **There is no strong evidence that `avg_rating` missingness depends on `day_of_week`.**

---

## Hypothesis Testing

We conducted a hypothesis test to determine whether the **number of ingredients** in a recipe is associated with its **average rating**.

### Research Question

> Do recipes with more ingredients tend to receive different average ratings than those with fewer ingredients?

---

### Hypotheses

- **Null Hypothesis (H₀)**: There is *no* difference in average ratings between recipes with more ingredients and those with fewer ingredients.
- **Alternative Hypothesis (H₁)**: There is a difference in average ratings between recipes with more ingredients and those with fewer ingredients.

### Test Design

- **Test Type**: Permutation test
- **Test Statistic**: Difference in mean average rating between recipes with more than the median number of ingredients vs. less than or equal to the median
- **Significance Level (α)**: 0.05

We first split the dataset into two groups:
- Group A: Recipes with `n_ingredients` > median
- Group B: Recipes with `n_ingredients` ≤ median

We computed the observed difference in their mean `avg_rating`, then shuffled the `n_ingredients` values 1000 times to create a null distribution of differences. The p-value was computed as the proportion of the absolute permuted differences that were greater than or equal to the observed difference.

<iframe
  src="assets/hyp.html"
  width="700"
  height="400"
  frameborder="0"
></iframe>

- **Observed Difference**: -0.004
- **p-value**: 0.370

Since the p-value is **greater than 0.05**, we **fail to reject the null hypothesis**. This means we do not have sufficient evidence to say that the number of ingredients in a recipe significantly affects its average rating.

----

## Framing a Prediction Problem

### Prediction Question

> Can we predict a recipe’s average rating using features available at the time of submission?

This prediction task builds directly on our initial question about what characteristics are associated with highly rated recipes. Instead of simply analyzing correlations, we aim to build a model that predicts `avg_rating`.

---

### Prediction Type

- **Type**: Regression
- **Response Variable**: `avg_rating`
  - `avg_rating` is a quantitative variable representing the average user rating of a recipe.


### Features Used

We restricted our features to those available at the time a recipe is submitted, rather than features not known at the time.

- `n_ingredients`: Number of ingredients in the recipe  
- `n_steps`: Number of instructions  
- `minutes`: Total prep time in minutes  
- `calories`, `protein_pdv`, `total_fat_pdv`, `sugar_pdv`, `sodium_pdv`: Nutritional values derived from the `nutrition` column  
- `description_length`: Number of characters in the recipe description  
- `is_weekend`: Whether the recipe was submitted on a weekend

### Evaluation Metric

We chose **Root Mean Squared Error (RMSE)** to evaluate our model because RMSEs are applicable for regression problems and penalize large errors more heavily than MAEs while providing an interpretable measure in the same units as the target variable.

---

## Baseline Model

We built a baseline regression model to predict a recipe’s average rating using features available at the time of submission.

---

### Features Used

The baseline model used the following **quantitative features**:

- `n_ingredients` (int): Number of ingredients  
- `n_steps` (int): Number of steps in the recipe  
- `minutes` (int): Total prep time  
- `calories` (float): Total calories  
- `protein_pdv` (float): Protein percent daily value  
- `total_fat_pdv` (float): Fat percent daily value  
- `sugar_pdv` (float): Sugar percent daily value  
- `sodium_pdv`(float): Sodium percent daily value

### Performance

- Baseline RMSE: 0.6360

The baseline model establishes a starting point for performance. Although this model can improve, it already captures some meaning from the features and establishes a benchmark to refer back to.

---

## Final Model

To improve upon our baseline, we engineered new features and used a more flexible model with hyperparameter tuning.

---

### New Features Added

We added two new features to capture additional information:

- **`description_length`**: Number of characters in the recipe’s `description` field. 
  - Users may have a preference for description length as longer descriptions may include more clear guidance and indicate expertise, building trust with the users.
- **`is_weekend`**: Boolean indicating whether the recipe was submitted on a Saturday or Sunday, derived from `submitted`.
  - Recipe creators may have more time on the weekends to carefully craft their dishes which may impact how users perceive it.

### Model and Pipeline

We replaced the linear model with a **Random Forest Regressor**, which can capture non-linear interactions and variable importance more effectively.

We performed **hyperparameter tuning** using `GridSearchCV`. Cross-validation was performed on the training set and the combination that minimized the RMSE was selected for the hyperparameters.

### Best Hyperparameters

- `n_estimators`: 100
- `max_depth`: 5

### Performance

- Final RMSE: 0.6348

Compared to our baseline, this model reduced error and demonstrated improved predictive power, particularly due to the new features and model flexibility. The final model performs better than the baseline, supporting the idea that features such as `description_length` and `is_weekend` help to explain user ratings. This model is more expressive and better suited for capturing subtle interactions between recipe attributes.

---

## Fairness Analysis

To assess whether our final model performs equitably across groups, we performed a fairness analysis comparing prediction error for recipes submitted on weekends vs. weekdays.

---

### Group Definitions

- **Group X**: Recipes submitted on a **weekend** (`is_weekend = 1`)
- **Group Y**: Recipes submitted on a **weekday** (`is_weekend = 0`)
- **Evaluation Metric**: RMSE (Root Mean Squared Error)
- **Test Statistic**: The *absolute difference* between the RMSE for each group

### Hypotheses

- **Null Hypothesis (H₀)**: The model’s RMSE is equal for weekend and weekday recipes.
- **Alternative Hypothesis (H₁)**: The model’s RMSE differs between weekend and weekday recipes.
- **Significance Level (α)**: 0.05

<iframe
  src="assets/perm_rmse2.html"
  width="700"
  height="400"
  frameborder="0"
></iframe>

- **Observed RMSE difference**: 0.0329
- **p-value**: 0.5230

Since the p-value is greater than 0.05, we **fail to reject the null hypothesis**. This suggests that our model does **not show statistically significant unfairness** in predictive accuracy between weekend and weekday recipes.
