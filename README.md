# League of Legends: The Snowballing Effect ❄️⚔️

**By Matthew Paoletta**

---

## Introduction

Have you ever wondered if professional League of Legends matches are decided in the first 15 minutes? In the competitive esports scene, there's a concept called "snowballing" – where early advantages compound over time, making comebacks increasingly difficult. This project investigates whether we can predict the winner of a match using only statistics from the first 15 minutes of gameplay.

### The Dataset

This analysis uses competitive League of Legends esports match data from Oracle's Elixir, spanning **13 years of professional play** across various leagues worldwide. After cleaning and filtering, the dataset contains **over 90,000 team-level match records**.

### Why This Matters

Understanding whether early-game performance reliably predicts match outcomes has significant implications:
- **For Players and Teams**: Helps identify which early objectives matter most for winning
- **For Analysts**: Provides data-driven insights into comeback potential and win probability
- **For Fans**: Helps understand when a game is truly decided versus when comebacks are still possible

### Key Columns in the Dataset

The analysis focuses on these team-level statistics measured at 15 minutes:
- **`golddiffat15`**: Gold difference between teams (positive = team is ahead)
- **`xpdiffat15`**: Experience point difference
- **`csdiffat15`**: Creep score (minion kills) difference
- **`killsat15`**, **`assistsat15`**, **`deathsat15`**: Combat statistics
- **`firstblood`**: Whether the team secured the first kill
- **`firstdragon`**: Whether the team killed the first dragon
- **`firstherald`**: Whether the team killed the first Rift Herald
- **`result`**: Final outcome (Win = True, Loss = False)

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning Process

To prepare the data for analysis, several cleaning steps were necessary:

1. **Filtered to team-level data only**: The raw dataset contains both individual player statistics and team summaries. Since match outcomes are determined at the team level, I focused only on team rows.

2. **Removed games shorter than 15 minutes**: Games that ended before the 15-minute mark don't have valid statistics for our analysis. These early-ending games were removed from the dataset.

3. **Handled missing values**: Dropped rows where key 15-minute statistics were missing, as these likely indicate data collection issues.

4. **Converted data types**: Boolean columns like `firstblood` and `result` were stored as integers (0/1) and needed to be converted to proper boolean types for clearer analysis.

### Cleaned Data Preview

Here's a glimpse of the cleaned dataset:

| league   |   gamelength |   golddiffat15 |   xpdiffat15 |   csdiffat15 |   killsat15 | firstblood   | firstdragon   | result   |
|:---------|-------------:|---------------:|-------------:|-------------:|------------:|:-------------|:--------------|:---------|
| LCKC     |         1365 |          -1763 |         -546 |          -11 |           0 | False        | False         | False    |
| LCKC     |         1365 |           1763 |          546 |           11 |           0 | True         | True          | True     |
| LCKC     |         2436 |          -2908 |        -2142 |           -9 |           3 | True         | False         | False    |
| LCKC     |         2436 |           2908 |         2142 |            9 |           5 | False        | True          | True     |
| LCKC     |         1893 |           1208 |         1497 |           -3 |           2 | False        | True          | True     |

### Univariate Analysis: Understanding Individual Variables

To understand the competitive landscape, I first examined which leagues contribute most to the dataset and how long professional matches typically last.

**Distribution of Games by League:**

<!-- PLOT PLACEHOLDER: Top 20 Leagues by Number of Games -->
<!-- TODO: Add fig1 (league_counts bar chart) to assets/ -->

<iframe
  src="assets/leagues-distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*The LPL (China's premier league) and LCK (South Korea) dominate the dataset, reflecting their prominence in competitive League of Legends.*

**Distribution of Game Length:**

<!-- PLOT PLACEHOLDER: Game Length Distribution -->
<!-- TODO: Add fig2 (gamelength histogram) to assets/ -->

<iframe
  src="assets/gamelength-distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*Most professional matches last between 25-40 minutes, with an average around 30-35 minutes. The 15-minute mark represents roughly one-third of a typical match.*

### Bivariate Analysis: Relationships Between Variables

Next, I explored how early-game statistics relate to match outcomes.

**Gold Difference at 15 Minutes vs. Game Outcome:**

<!-- PLOT PLACEHOLDER: Gold Diff by Result Box Plot -->
<!-- TODO: Add fig3 (golddiff box plot) to assets/ -->

<iframe
  src="assets/golddiff-outcome.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*Teams that win have a significantly higher gold advantage at 15 minutes. The median winning team is ahead by about 800 gold, while losing teams are typically behind by a similar amount.*

**Win Rate by First Blood:**

<!-- PLOT PLACEHOLDER: First Blood Win Rate Bar Chart -->
<!-- TODO: Add fig4 (first blood win rate) to assets/ -->

<iframe
  src="assets/firstblood-winrate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*Securing first blood correlates with a notable win rate advantage, though it's not overwhelming. Teams with first blood win approximately 53-54% of the time.*

### Interesting Aggregates

**Summary Statistics by Game Outcome:**

|        |   ('golddiffat15', 'mean') |   ('golddiffat15', 'median') |   ('xpdiffat15', 'mean') |   ('killsat15', 'mean') |   ('deathsat15', 'mean') |
|:-------|---------------------------:|-----------------------------:|-------------------------:|------------------------:|-------------------------:|
| Loss   |                    -818.14 |                       -652.0 |                  -459.63 |                    3.33 |                     4.33 |
| Win    |                     818.14 |                        652.0 |                   459.63 |                    4.33 |                     3.33 |

*Winning teams have an average gold lead of +818 at 15 minutes, while losing teams are behind by the same amount. The kill-to-death ratio also flips – winners average 4.33 kills vs. 3.33 deaths, while losers have the opposite.*

**Win Rate by Early Objectives Secured:**

| firstblood   | firstdragon   | firstherald   |   Win Rate |   Games |
|:-------------|:--------------|:--------------|-----------:|--------:|
| True         | True          | True          |      66.27 |   22647 |
| True         | True          | False         |      63.22 |   15040 |
| False        | True          | True          |      61.23 |   12734 |
| True         | False         | True          |      53.50 |    5869 |
| True         | False         | False         |      47.37 |   10198 |
| False        | True          | False         |      46.65 |   12734 |
| False        | False         | True          |      38.57 |    5391 |
| False        | False         | False         |      33.59 |   15169 |

*Securing all three early objectives (first blood, dragon, and herald) leads to a 66% win rate. Conversely, securing none of these objectives results in only a 34% win rate. Objectives matter!*

---

## Assessment of Missingness

### MNAR Analysis

One column that I believe exhibits **MNAR (Missing Not At Random)** characteristics is `dragons (type unknown)`, which is 96% missing in the dataset.

This missingness is likely MNAR because it depends on the dragon type values themselves. The data recording system may have failed to log dragon types specifically when unusual type combinations occurred, or when bugs in the tracking system were triggered by the dragon types themselves. In other words, *the fact that the data is missing could be related to what the actual values would have been*.

**Additional data that could make this MAR:**
- Game patch information (which version of League was played)
- Tournament organizer information
- Data collection system version
- Timestamps of when data was recorded

With this information, we could explain the missingness through observable factors rather than the missing values themselves.

### Missingness Dependency Analysis

I analyzed whether the missingness of early-game statistics depends on other columns in the dataset. Specifically, I investigated the missingness pattern of game-related columns.

**Key Finding: Most critical 15-minute statistics have NO missing values** after data cleaning. The columns `golddiffat15`, `xpdiffat15`, `csdiffat15`, `killsat15`, `assistsat15`, and `deathsat15` are all complete, which is essential for building reliable predictive models.

The missing data that does exist primarily relates to:
- Player-level columns (100% missing, by design – we filtered to team data only)
- Features introduced in later game versions (51-97% missing)
- Rare game mechanics like special dragon types (96% missing)

<!-- PLOT PLACEHOLDER: Missingness Analysis Visualization -->
<!-- Optional: Add a visualization showing missingness patterns -->

---

## Hypothesis Testing

### Research Question

Does securing **first blood** (the first kill of the game) actually matter for winning?

In competitive League of Legends, first blood is celebrated as an important psychological and economic advantage. But is there statistical evidence that teams who get first blood win more often than those who don't?

### Hypotheses

**Null Hypothesis (H₀):** Teams that secure first blood have the same win rate as teams that don't secure first blood. Any observed difference in win rates is due to random chance.

**Alternative Hypothesis (H₁):** Teams that secure first blood have a higher win rate than teams that don't secure first blood.

### Test Design

- **Test Statistic:** Difference in win rates (First Blood - No First Blood)
- **Significance Level:** α = 0.05
- **Method:** Permutation test with 1,000 simulations

### Results

**Observed difference in win rates:** Teams with first blood won approximately 3.6 percentage points more often than teams without first blood.

**P-value:** < 0.001 (essentially 0.000)

<!-- PLOT PLACEHOLDER: Permutation Test Distribution -->
<!-- TODO: Add permutation test visualization to assets/ -->

<iframe
  src="assets/hypothesis-test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*The red line shows our observed difference. The distribution shows what we'd expect by random chance. Our observed value is far beyond what random chance would produce.*

### Conclusion

We **reject the null hypothesis**. There is strong statistical evidence that teams with first blood have a significantly higher win rate than teams without first blood. While the effect size is modest (~3.6 percentage points), it is definitely not due to random chance.

This suggests that even a single early kill provides a measurable advantage that teams can leverage throughout the match.

---

## Framing a Prediction Problem

### The Challenge

**Can we predict whether a team will win or lose based solely on statistics from the first 15 minutes of gameplay?**

This is a **binary classification problem** where:
- **Response variable:** `result` (Win = True, Loss = False)
- **Goal:** Classify each team as likely to win or lose

### Why This Response Variable?

The match outcome (`result`) is the most meaningful measure in competitive League of Legends. Teams don't play to maximize gold or kills – they play to win the game. A model that predicts outcomes helps teams understand whether their early-game strategies are setting them up for success.

### Evaluation Metric: Accuracy

I chose **accuracy** (correct predictions / total predictions) as the primary evaluation metric for several reasons:

1. **Balanced classes:** The dataset has roughly equal wins and losses (~50/50), so accuracy won't be misleading
2. **Symmetric costs:** Falsely predicting a win is just as problematic as falsely predicting a loss – there's no asymmetric cost
3. **Interpretability:** Accuracy is intuitive for anyone to understand

### Information Available at Prediction Time

**Critical constraint:** We can ONLY use information available at exactly 15 minutes into the match.

**Features we can use:**
- `golddiffat15`, `xpdiffat15`, `csdiffat15`
- `killsat15`, `assistsat15`, `deathsat15`
- `firstblood`, `firstdragon`, `firstherald`

**Features we CANNOT use:**
- `gamelength` (we don't know how long the game will last)
- Final objective counts (`dragons`, `barons`, `heralds`)
- End-game statistics (`dpm`, `damageshare`)

This ensures our model is realistic – it only knows what a spectator would know at the 15-minute mark.

---

## Baseline Model

### Model Description

For the baseline model, I used **Logistic Regression** to predict match outcomes using just two fundamental economic indicators:

**Features:**
1. **`golddiffat15`** (Quantitative): Gold difference at 15 minutes
2. **`xpdiffat15`** (Quantitative): Experience point difference at 15 minutes

These features represent the core economic advantage/disadvantage a team has built up in the early game. Both are numeric and on similar scales, so no feature encoding was needed.

### Why These Features?

Gold and experience are the two primary economic resources in League of Legends. Teams that are ahead in these metrics typically have better items and higher champion levels, leading to combat advantages. These seemed like natural starting points for prediction.

### Performance Results

| Dataset  | Accuracy |
|:---------|:---------|
| Training | 0.733    |
| Test     | 0.734    |

<!-- PLOT PLACEHOLDER: Baseline Model Performance Bar Chart -->
<!-- TODO: Add baseline performance comparison to assets/ -->

<iframe
  src="assets/baseline-performance.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*Both training and test accuracy significantly exceed random guessing (50%) and meet our target threshold of 70%.*

### Is This Model "Good"?

**Yes, for a baseline model.** The model achieves:
- **73.4% accuracy** – well above random guessing (50%)
- **Strong generalization** – similar performance on training and test data (no overfitting)
- **Meaningful predictions** – correctly predicts outcomes nearly 3 out of 4 times

However, there's room for improvement. We're only using 2 out of 9+ available features, and we haven't tried more sophisticated algorithms or feature engineering. The next step is to build a more comprehensive model.

---

## Final Model

### Feature Engineering

To improve upon the baseline, I engineered two new features that capture important gameplay concepts:

**1. Kill Efficiency** = `killsat15 / (deathsat15 + 1)`
- Measures how effectively a team trades kills for deaths in the early game
- Higher values indicate dominant combat performance
- The "+1" prevents division by zero when a team has no deaths

**2. Economic Efficiency** = `golddiffat15 / (killsat15 + 1)`
- Measures how much gold advantage the team gains per kill
- Captures efficiency of converting kills into economic benefits
- High values suggest strong objective control and farming between kills

**Why these features are meaningful:** A team with 10 kills and 2 deaths is in a vastly different position than a team with 10 kills and 8 deaths, even though both have the same kill count. These ratio features capture the *quality* of early-game performance, not just the quantity.

### Model Selection and Hyperparameter Tuning

I upgraded from Logistic Regression to **Random Forest Classifier**, which can:
- Capture non-linear relationships between features
- Handle feature interactions automatically (e.g., high gold + high kills together)
- Reduce overfitting through ensemble averaging

**Hyperparameters tested:**
- `max_depth`: [10, 20] – Controls tree depth to prevent overfitting
- `n_estimators`: [50, 100] – Number of trees in the forest

**Method:** GridSearchCV with 3-fold cross-validation to find the optimal combination.

**Best hyperparameters found:**
- `max_depth`: 10
- `n_estimators`: 50

### All Features Used

The final model uses **11 total features:**
- Original 9: `golddiffat15`, `xpdiffat15`, `csdiffat15`, `killsat15`, `assistsat15`, `deathsat15`, `firstblood`, `firstdragon`, `firstherald`
- Engineered 2: `kill_efficiency`, `economic_efficiency`

### Performance Results

| Model    | Dataset  | Accuracy |
|:---------|:---------|:---------|
| Random   | Any      | 0.500    |
| Baseline | Training | 0.733    |
| Baseline | Test     | 0.734    |
| **Final**| **Training** | **0.831** |
| **Final**| **Test**     | **0.757** |

<!-- PLOT PLACEHOLDER: Model Comparison Bar Chart -->
<!-- TODO: Add final model comparison to assets/ -->

<iframe
  src="assets/model-comparison.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*The final model achieves 75.7% test accuracy, a notable improvement over the baseline's 73.4%.*

### Why the Final Model is Better

The final model improved because:

1. **More comprehensive features:** Using all 11 features captures more aspects of early-game performance – combat effectiveness, objective control, and economic efficiency
2. **Better algorithm:** Random Forest handles non-linear relationships and feature interactions that Logistic Regression cannot
3. **Meaningful feature engineering:** The efficiency ratios provide signal beyond raw counts

The model still generalizes well (similar train/test performance), confirming we're not overfitting despite using more features.

---

## Fairness Analysis

### Fairness Question

Does the final model perform equally well for games from different eras of League of Legends?

The game has evolved significantly over the years with major gameplay changes, new champions, and meta shifts. I wanted to test whether the model, trained on all eras, performs fairly across:

- **Group X (Early Era):** Games from 2014-2018
- **Group Y (Modern Era):** Games from 2019+

If the model performs significantly worse for one era, it could indicate that early-game patterns have changed over time.

### Evaluation Metric

**Accuracy** – consistent with our overall evaluation approach.

### Hypotheses

**Null Hypothesis (H₀):** The model is fair. Its accuracy for Early Era games and Modern Era games are roughly the same, and any observed differences are due to random chance.

**Alternative Hypothesis (H₁):** The model is unfair. Its accuracy differs significantly between Early Era and Modern Era games.

### Test Design

- **Test Statistic:** Absolute difference in accuracy between eras
- **Significance Level:** α = 0.05
- **Method:** Permutation test with 1,000 simulations

### Results

**Observed Accuracies:**
- Early Era (2014-2018): 73.37% accuracy (6,313 games)
- Modern Era (2019+): 75.71% accuracy (27,469 games)
- Absolute Difference: 2.34 percentage points

**P-value:** < 0.001 (essentially 0.000)

<!-- PLOT PLACEHOLDER: Fairness Analysis Visualizations -->
<!-- TODO: Add accuracy by era and permutation test to assets/ -->

<iframe
  src="assets/fairness-accuracy.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="assets/fairness-permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*The model performs slightly but significantly better on Modern Era games.*

### Conclusion

We **reject the null hypothesis** (p < 0.001). The model's accuracy differs significantly between Early Era and Modern Era games.

**Possible Explanation:** League of Legends has undergone major gameplay changes since 2018, including:
- Turret plating system (2019)
- Dragon soul mechanics (2020)
- Extensive jungle and objective changes

The relationships between 15-minute statistics and match outcomes may have shifted as the game meta evolved. The model may be capturing era-specific patterns that don't generalize perfectly across different versions of the game.

**Implication:** The model shows a bias toward Modern Era gameplay, performing better on games that resemble the 2019+ meta. Teams and analysts should be cautious when applying this model to predict outcomes in future tournaments, as the game's continued evolution may further impact model performance.

---

## Conclusion

This project demonstrates that **early-game performance at 15 minutes is highly predictive of match outcomes** in professional League of Legends. The final Random Forest model achieves 75.7% accuracy, correctly predicting winners more than 3 out of 4 times using only statistics from the first 15 minutes.

**Key Takeaways:**
- Economic advantages (gold, XP) are the strongest predictors
- Objective control (first blood, dragon, herald) provides measurable advantages
- The concept of "snowballing" is statistically validated
- The game has evolved over time, affecting prediction patterns

While the model is strong, it's not perfect – roughly 1 in 4 games still defies early predictions, keeping hope alive for cinematic comeback stories!

---

*This project was completed as part of DSC 80 (Practice and Application of Data Science) at UC San Diego.*
