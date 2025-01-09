

# Systolic Blood Pressure Analysis
# Author
**Rajin Panthee**

## Overview
This project analyzes data from the Jackson Heart Study (JHS) to investigate factors influencing systolic blood pressure (SBP). The analysis involves modeling SBP as a function of age, education, and body mass index (BMI), assessing model assumptions, and visualizing the results.

## Objectives
- Understand the relationship between SBP and predictor variables (age, education, and BMI).
- Assess the statistical significance of predictors.
- Visualize the results to aid interpretation.

## Analysis Steps

### 1. Importing Data
- **Task**: Load the JHS dataset, which is in SAS format.
- **Why**: To access the data for analysis in R, we need to import it into a compatible format.
- **Tool**: `haven` package in R.
- **Code**:
  ```r
  library(haven)
  jhs_data <- read_sas("C:\Users\rajpa\Downloads\analysis1.sas7bdat")
  ```

### 2. Initial Model: SBP as a Function of Age, Education, and BMI
- **Objective**: Fit a generalized linear model (GLM) with SBP as the dependent variable and age, education, and BMI as predictors.
- **Why**: To quantify the relationship between SBP and the predictors and evaluate their significance.
- **Key Outputs**:
  - Coefficients
  - Statistical significance
  - Confidence intervals
- **Code**:
  ```r
  m1 <- glm(sbp ~ age + HSgrad + BMI, data = jhs_data, family = gaussian)
  summary(m1)
  confint(m1)
  ```

### 3. Model Assessment
- **Significance Testing**: Compare the full model with a null model using ANOVA.
  - **Why**: To test if the predictors collectively explain a significant portion of the variance in SBP.
- **Code**:
  ```r
  full <- glm(sbp ~ age + HSgrad + BMI, data = data_new, family = gaussian)
  reduced <- glm(sbp ~ 1, data = data_new, family = gaussian)
  anova(reduced, full, test = "F")
  ```

- **Identify Outliers**:
  - **Why**: To check for data points that may disproportionately influence the model.
  ```r
  dataset <- data_new %>%
    mutate(outlier = if_else(abs(rstandard(m1)) > 2.5, "Suspected", "Not Suspected"))
  outlier_count <- dataset %>% count(outlier)
  ```

- **Assess Multicollinearity**:
  - **Why**: To ensure predictors are not highly correlated, which could bias the model.
  ```r
  car::vif(m1)
  ```

### 4. Visualizing the Results
- **Task**: Create scatter plots of SBP against age, with predicted values for different age groups overlaid.
- **Why**: To visually assess the relationship between SBP and predictors and interpret the model results.
- **Code**:
  ```r
  dataset %>%
    ggplot(aes(x = age)) +
    geom_point(aes(y = sbp)) +
    geom_line(aes(y = sbp_hat_age30), color = "blue") +
    geom_line(aes(y = sbp_hat_age40), color = "purple") +
    geom_line(aes(y = sbp_hat_age50), color = "green") +
    theme_bw()
  ```

### 5. Extended Model: Categorical Health Status
- **Objective**: Refit the model using health status as a categorical variable derived from BMI.
- **Why**: To evaluate the impact of health categories (poor, intermediate, ideal) on SBP for better interpretability.
- **Task**:
  - Create health status categories (`poor_health`, `intermediate_health`, `ideal_health`).
  - Fit a GLM with age, education, and health status as predictors.
- **Code**:
  ```r
  jhs_data <- jhs_data %>%
    mutate(health = case_when(BMI3cat == 0 ~ "poor_health",
                              BMI3cat == 1 ~ "intermediate_health",
                              BMI3cat == 2 ~ "ideal_health")) %>%
    dummy_cols(select_columns = c("health"))

  m2 <- glm(sbp ~ age + HSgrad + health, data = jhs_data, family = gaussian)
  summary(m2)
  ```

### 6. Final Visualization
- **Task**: Visualize SBP predictions for different health categories and ages.
- **Why**: To communicate the results effectively to a broader audience.
- **Code**:
  ```r
  dataset %>%
    ggplot(aes(x = age)) +
    geom_point(aes(y = sbp)) +
    geom_smooth(aes(y = sbp_hat_age30), color = "blue", se = FALSE) +
    geom_smooth(aes(y = sbp_hat_age40), color = "purple", se = FALSE) +
    geom_smooth(aes(y = sbp_hat_age50), color = "green", se = FALSE) +
    theme_minimal()
  ```

## Key Findings
- **Age**: A significant predictor of SBP, with SBP increasing as age increases.
- **Education**: No statistically significant impact on SBP.
- **BMI**: Higher BMI is associated with increased SBP. Health categories provide additional insights, with poor health linked to significantly higher SBP compared to ideal health.

## Files
- `sbp_analysis.qmd`: Contains the complete analysis code and documentation.
- `README.md`: Provides an overview of the project.

## How to Run
1. Ensure you have R and the required libraries installed (`haven`, `dplyr`, `ggplot2`, `fastDummies`, `car`).
2. Place the data file (`analysis1.sas7bdat`) in the specified directory.
3. Run the code chunks in the `sbp_analysis.qmd` file to reproduce the analysis.

