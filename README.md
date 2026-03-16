# League of Legends: The Snowballing Effect ❄️⚔️

**By Matthew Paoletta**

---

## Introduction

Can you predict who will win a professional League of Legends match just 15 minutes into the game? This project explores the "snowballing effect" – where early advantages compound over time, making comebacks harder and harder.

**The Dataset:** 13 years of competitive League of Legends data from Oracle's Elixir, with over **90,000 team-level match records** from leagues worldwide.

**The Question:** Can early-game stats predict match winners?

**Why it matters:**
- **Teams** learn which early objectives drive wins
- **Analysts** can calculate real-time win probabilities
- **Fans** know when comebacks are still possible

**Key stats at 15 minutes:**
- `golddiffat15` – Gold difference (how much richer one team is)
- `xpdiffat15` – Experience difference (level advantages)
- `csdiffat15` – Farm difference (minion kills)
- `killsat15`, `assistsat15`, `deathsat15` – Combat stats
- `firstblood`, `firstdragon`, `firstherald` – Early objectives
- `result` – Did the team win? (our prediction target)

---

## Data Cleaning and Exploratory Data Analysis

### Cleaning the Data

Four key steps prepared the data for analysis:

1. **Kept only team-level data** – The raw data has stats for individual players AND team totals. We only need the team totals.

2. **Removed super short games** – Games under 15 minutes don't have 15-minute stats (duh!).

3. **Dropped rows with missing 15-minute stats** – If key data is missing, we can't use that game.

4. **Fixed data types** – Converted 0/1 integers to True/False for clarity.

### Cleaned Data Preview

Here's a glimpse of the cleaned dataset:

| league   |   gamelength |   golddiffat15 |   xpdiffat15 |   csdiffat15 |   killsat15 | firstblood   | firstdragon   | result   |
|:---------|-------------:|---------------:|-------------:|-------------:|------------:|:-------------|:--------------|:---------|
| LCKC     |         1365 |          -1763 |         -546 |          -11 |           0 | False        | False         | False    |
| LCKC     |         1365 |           1763 |          546 |           11 |           0 | True         | True          | True     |
| LCKC     |         2436 |          -2908 |        -2142 |           -9 |           3 | True         | False         | False    |
| LCKC     |         2436 |           2908 |         2142 |            9 |           5 | False        | True          | True     |
| LCKC     |         1893 |           1208 |         1497 |           -3 |           2 | False        | True          | True     |

### What the Data Shows

**Distribution of Games by League:**

<iframe
  src="assets/leagues-distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*Asian leagues (LPL, LCK) dominate the dataset – they're the powerhouses of competitive League.*

**How Long Do Matches Last?**

<iframe
  src="assets/gamelength-distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*Most matches last 25-40 minutes. The 15-minute mark is about one-third of the way through.*

**Does Early Gold Lead = Win?**

<iframe
  src="assets/golddiff-outcome.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*YES! Winning teams are typically ahead by ~800 gold at 15 minutes, while losing teams are behind by about the same.*

**First Blood = Victory?**

<iframe
  src="assets/firstblood-winrate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*Getting the first kill helps, but it's not overwhelming – about 53-54% win rate vs. 46-47% without.*

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

### MNAR (Missing Not At Random)

**`dragons (type unknown)`** is 96% missing – likely **MNAR**. Why? The data recording system may have *failed to log dragon types specifically when unusual combinations occurred* or when bugs were triggered by the dragon types themselves. The missingness depends on what the values would have been.

**What could make this MAR instead?** If we had game patch info, tournament organizer details, or data collection system versions, we could explain the missingness through observable factors.

### Missingness Dependency: Permutation Tests

To understand whether missingness patterns are random or systematic, I ran two formal permutation tests on **`turretplates`** (51% missing):

**Test 1: Does `turretplates` missingness depend on `year`?**

- **Expected:** YES – turret plates were introduced in 2019, so older games shouldn't have this data
- **Test statistic:** Total Variation Distance (TVD) between year distributions
- **Result:** p < 0.001 → **DOES depend on year**
- **Conclusion:** Missingness is MAR (Missing At Random) – entirely explained by when the game was played

<iframe
  src="assets/missingness-year.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Test 2: Does `turretplates` missingness depend on `result` (win/loss)?**

- **Expected:** NO – whether data is missing shouldn't relate to who won
- **Test statistic:** Absolute difference in missingness rates
- **Result:** p > 0.05 → **Does NOT depend on result**
- **Conclusion:** Missingness is unrelated to game outcomes

<iframe
  src="assets/missingness-result.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

These tests confirm that features missing due to game evolution (like turret plates) can be safely handled – the missingness doesn't bias our win/loss predictions.

**Bottom line:** All critical 15-minute stats (`golddiffat15`, `xpdiffat15`, `csdiffat15`, `killsat15`, `assistsat15`, `deathsat15`) are 100% complete. Our prediction model has the data it needs.

---

## Hypothesis Testing

### The Question

Does getting **first blood** (the first kill) actually help teams win?

### Hypotheses

**Null Hypothesis:** First blood doesn't matter – the win rate is the same with or without it.

**Alternative Hypothesis:** First blood helps – teams with it win more often.

### Test Design

- **Test Statistic:** Difference in win rates (First Blood wins - No First Blood wins)
- **Significance Level:** α = 0.05
- **Method:** Permutation test (1,000 simulations)

### Results

**Observed difference:** Teams with first blood won 3.6 percentage points more often.

**P-value:** < 0.001

<iframe
  src="assets/hypothesis-test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*The red line is our observed difference. The blue histogram shows what random chance would produce. Our value is way outside that range.*

### Conclusion

**We reject the null hypothesis.** First blood DOES matter – teams with first blood win significantly more often. The effect is real, though modest (~3.6%) – it helps, but doesn't guarantee victory.

---

## Framing a Prediction Problem

**The Challenge:** Can we predict who will win using only the first 15 minutes of gameplay?

**Type:** Binary classification (Win or Loss)

**Response Variable:** `result` – Did the team win?

**Evaluation Metric:** Accuracy (% of correct predictions)

**Why accuracy?** The dataset is perfectly balanced (50% wins, 50% losses), and both types of mistakes (false win/false loss) matter equally.

**Features we can use:** Only stats from the first 15 minutes – `golddiffat15`, `xpdiffat15`, `csdiffat15`, `killsat15`, `assistsat15`, `deathsat15`, `firstblood`, `firstdragon`, `firstherald`

**Features we CAN'T use:** Anything from after 15 minutes like final game length or end-game objectives

---

## Baseline Model

### The Simple Approach

For the baseline model, I used **Logistic Regression** to predict match outcomes using just two fundamental economic indicators:

**Features:**
1. **`golddiffat15`** – Gold difference at 15 minutes (already numeric)
2. **`firstblood`** – Did the team get the first kill? (converted True/False to 1/0)

Why these two? They're the most obvious early advantages: gold = better items, first blood = early momentum.

### Performance

| Dataset  | Accuracy |
|:---------|:---------|
| Training | 0.733    |
| Test     | 0.734    |

<iframe
  src="assets/baseline-performance.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*Both training and test accuracy beat random guessing (50%) and hit our 70% target.*

### Is This Good?

**Yes, for a start.** The model gets it right 73% of the time – nearly 3 out of 4 matches. It doesn't overfit (train and test scores are similar). But we're only using 2 of 9+ available features, so there's definitely room for improvement.

---

## Final Model

### Making It Better

I added two smart features that capture how *efficiently* teams play:

**1. Kill Efficiency** = `killsat15 / (deathsat15 + 1)`  
How many kills per death? A team with 10 kills and 2 deaths (ratio = 5.0) is crushing it. A team with 10 kills and 8 deaths (ratio = 1.1) is barely hanging on.

**2. Economic Efficiency** = `golddiffat15 / (killsat15 + 1)`  
How much gold advantage per kill? High values mean the team is securing objectives and farming well, not just fighting.

I also upgraded to a **Random Forest** algorithm (50 trees, max depth 10), which handles complex patterns better than simple models.

### All Features Used (11 total)

- Original 9: `golddiffat15`, `xpdiffat15`, `csdiffat15`, `killsat15`, `assistsat15`, `deathsat15`, `firstblood`, `firstdragon`, `firstherald`
- Engineered 2: `kill_efficiency`, `economic_efficiency`

### Performance

| Model    | Dataset  | Accuracy |
|:---------|:---------|:---------|
| Random   | Any      | 0.500    |
| Baseline | Training | 0.733    |
| Baseline | Test     | 0.734    |
| **Final**| **Training** | **0.831** |
| **Final**| **Test**     | **0.757** |

<iframe
  src="assets/model-comparison.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*The final model gets 75.7% accuracy – up from 73.4% in the baseline.*

### Why It's Better

Three reasons:
1. **Uses all 11 features** – captures combat, objectives, AND efficiency
2. **Random Forest algorithm** – handles complex patterns that simple models miss
3. **Smart feature engineering** – kill/economic efficiency ratios reveal team performance quality

Train and test scores are similar, so we're not overfitting.

---

## Fairness Analysis

### The Question

Does the model work equally well for modern games (2022+) vs. older games (pre-2022)?

League of Legends changes constantly – new champions, items, balance patches. I tested whether the model performs fairly across different eras:

- **Early Era:** 2014-2018
- **Modern Era:** 2019+

### Hypotheses

**Null Hypothesis (H₀):** The model is fair – accuracy is similar for both eras, and any difference is just random chance.

**Alternative Hypothesis (H₁):** The model is unfair – accuracy differs significantly between eras.

### Test Design

- **Test Statistic:** Absolute difference in accuracy between eras
- **Significance Level:** α = 0.05
- **Method:** Permutation test (1,000 simulations)

### Results

**Observed Accuracies:**
- Early Era: 73.37%
- Modern Era: 75.71%
- Difference: 2.34 percentage points

**P-value:** < 0.001

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

*The model performs slightly better on modern games.*

### Conclusion

**We reject the null hypothesis** (p < 0.001). The model IS biased toward modern games.

**Why?** League changed massively after 2018:
- Turret plating system (2019)
- Dragon soul mechanics (2020)
- Jungle/objective reworks

The model learned patterns from modern League, so it works better on modern data. For future predictions, be aware that continued game evolution may impact accuracy.

---

## Wrap-Up

**Early-game performance at 15 minutes strongly predicts match outcomes** in pro League. The final model gets it right 75.7% of the time – more than 3 out of 4 matches.

**Key insights:**
- Gold and XP advantages are the strongest predictors
- Objectives (first blood, dragon, herald) provide measurable edges
- The "snowballing effect" is statistically real
- The game evolves over time, affecting prediction patterns

But remember: 1 in 4 games still defy early predictions – comebacks DO happen!

---

*This project was completed as part of DSC 80 (Practice and Application of Data Science) at UC San Diego.*
