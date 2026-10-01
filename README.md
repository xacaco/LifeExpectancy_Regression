# Life Expectancy and Its Determinants: A Multiple Regression Analysis

A statistical modelling project in R that examines how health, economic and social indicators relate to national **life expectancy**, using the WHO life expectancy data published on Kaggle.

The analysis is contained in a single R Markdown notebook, `LifeExp_Regression.Rmd`, and covers data preparation, descriptive statistics, model building (including non-linear and interaction terms), variable selection, regression diagnostics and cross-validation.

---

## Data

- **Source:** [Life Expectancy (WHO) on Kaggle](https://www.kaggle.com/datasets/kumarajarshi/life-expectancy-who). The dataset compiles country-level health and economic indicators from WHO and related sources, with one row per country per year.
- **Expected file:** the notebook reads a CSV named `Life Expectancy Data (2).csv` from the working directory. The data file is not included in this repository; download it from Kaggle and place it next to the notebook (rename it, or edit the `read.csv()` call, if your file name differs).

### Variables used

| Role | Variable | Notebook name |
| --- | --- | --- |
| Outcome | Life expectancy (years) | `Life.expectancy` |
| Predictor | Development status (developed / developing) | `Status` |
| Predictor | Alcohol consumption | `Alcohol` |
| Predictor | Total health expenditure | `Total.expenditure` |
| Predictor | HIV/AIDS deaths | `HIV.AIDS` |
| Predictor | Population | `Population` |
| Predictor | Income composition of resources (Human Development Index component), aliased as `ICR` | `Income.composition.of.resources` |
| Predictor | GDP | `GDP` |
| Predictor | Years of schooling | `Schooling` |
| Predictor | Infant deaths | `infant.deaths` |
| Descriptive grouping | Continent (added manually, see below) | `Continent` |

---

## What was done

### 1. Data preparation
- Loaded the raw data and filtered it to the years 2000–2015, sorted by country and year.
- Built a **country-to-continent lookup table** (Africa, Asia, Europe, North America, Oceania, South America) by listing countries manually, then merged it with the WHO data by country name.
- Restricted the merged data to a single year (**2009**) to obtain a cross-sectional dataset with one observation per country.
- Selected the outcome, the nine predictors and the grouping variables into the final analysis data frame, `project_df`. Rows with missing values are dropped (`na.omit`) when models are fitted.

### 2. Descriptive analysis
- Summary table (mean, mode, Q1, median, Q3, range, standard deviation, 10th and 90th percentiles) produced with `summarytools` and rendered as an HTML table with `kableExtra`.
- Histograms of every numeric variable.
- Boxplot of life expectancy by continent and a histogram of life expectancy.

### 3. Model building
- **Full linear model:** life expectancy regressed on all nine predictors, with partial effects visualized using the `effects` package.
- **Non-linear terms:** a second-degree orthogonal polynomial for `HIV.AIDS`, with the polynomial term's effect plotted.
- **Interaction terms:** an `Alcohol × ICR` interaction, fitted alone and combined with the polynomial term, with the interaction effect plotted.
- **Model comparison:**
  - Nested models compared with F-tests (`anova`) against a null (intercept-only) model.
  - Non-nested models (polynomial vs. interaction) compared with **AIC** and adjusted R².

### 4. Variable selection
- **Best-subset selection** with `leaps::regsubsets`, evaluated with adjusted R², Mallows' Cp and BIC across model sizes.
- **Backward stepwise selection** using the BIC penalty, starting from the polynomial model, to obtain a more parsimonious final model (`xbestmod`).
- Model fit summarized with adjusted R² and the coefficient of variation of the residual error.
- A **standardized version** of the final model (all numeric variables scaled) was fitted to put coefficients on a comparable scale.

### 5. Regression diagnostics (final model)
| Assumption / check | Methods |
| --- | --- |
| Multicollinearity | Variance inflation factors (`car::vif`) |
| Linearity | Residuals-vs-fitted plot, Ramsey RESET test (`lmtest::resettest`), component-plus-residual plots (`car::crPlots`) |
| Normality of residuals | Histogram, Q–Q plot, Shapiro–Wilk test, Kolmogorov–Smirnov test |
| Homogeneity of variance | Scale-location plot, Breusch–Pagan test (on the raw and log-transformed outcome) |
| Independence of errors | Residual autocorrelation function (ACF), Durbin–Watson test with bootstrapped p-values |
| Outliers, leverage and influence | Studentized residuals, hat values (2(k+1)/n and 3(k+1)/n thresholds), Cook's distance (4/n threshold), influence plots, added-variable plots, Bonferroni outlier test |

### 6. Predictive validation
- **10-fold cross-validation** (fixed seed) of best-subset models of each size, using a custom `predict` method for `regsubsets` objects.
- Mean cross-validated MSE computed per model size to identify the size with the lowest prediction error, followed by refitting on the full data to extract coefficients.

---
### Author

Xavier Calvet
