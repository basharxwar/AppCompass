[README.md](https://github.com/user-attachments/files/34424120/README.md)
# AppCompass: which Android apps become hits?

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/basharxwar/AppCompass/blob/main/AppCompass.ipynb)

**AppCompass** is a machine learning project on the Google Play Store. We ask one question: **what makes an Android app successful?** We call an app *popular* when it has **1,000,000 installs or more**, and we train models that predict which apps are popular.

> If your repository has another name or branch, change the link in the badge above.

## Results at a glance

| | |
|---|---|
| Data | 10,841 apps from the Google Play Store (2018), 9,656 after cleaning |
| Target | popular = 1,000,000 installs or more (35.2% of the apps) |
| Final model | Random Forest (no depth limit, at least 3 apps per leaf) |
| Cross-validation (5 folds) | F1 0.646, AUC 0.843, accuracy 0.777 |
| Test set (used once) | F1 0.659, AUC 0.843, accuracy 0.783 (a model that always says "not popular" gets 0.648) |
| What it finds | 404 of the 680 popular apps in the test set, and it is right 74% of the time when it says "popular" |
| Main drivers | size, days since the last update, category |

## What is in this repository

| File | What it is |
|---|---|
| `AppCompass.ipynb` | The whole project in one Colab notebook, from the raw file to the final results |
| `README.md` | This file |
| `AppCompass_slides.pdf` | The final presentation [add the file] |
| `googleplaystore_clean.csv` | The clean data that the notebook saves (optional, see "Data and license") |

## The story of the notebook

1. **Load the data and look at it:** 10,841 rows, 1,474 missing ratings, 1,181 repeated app names. A data dictionary explains every column.
2. **Clean the data.** Every step is a decision with a reason. We removed 1,185 rows (10.9%): 1 broken row, 2 "Unrated", 1 with no Type and 1,181 repeated apps. 1,461 missing ratings were filled with the median and marked in a column.
3. **Explore the data.** Ratings are not reliable for apps with few reviews, installs are concentrated in a few categories, and paid apps reach about 100 times fewer users.
4. **Prepare the data.** Target, seven features with reasons, a split first, a pipeline so nothing leaks from the test set, and a readiness check (7 columns became 33 numbers, no missing values, no overlap between training and test).
5. **Train and compare models:** a baseline, Logistic Regression, KNN, a Decision Tree and a Random Forest, with the same 5-fold cross-validation.
6. **Check the final model:** test set, ROC curve, thresholds, single predictions, what the model uses, learning curve, errors and an experiment.
7. **Extra:** K-Means groups the apps into 5 segments. The share of popular apps goes from 3% to 59% between the segments.
8. **Summary,** challenges and next steps.

## How this covers the milestones

| Milestone | What it asked for | Where it is |
|---|---|---|
| 2: Data cleaning | Find the problems, fix them with documented decisions, show before and after | Chapters 1 and 2 |
| 3: Exploration and preprocessing | Charts with insights, correlation analysis, features with reasons, a leak-free pipeline, a readiness check | Chapters 3 and 4 |
| 4: Machine learning | Models and why, training and evaluation, a fair comparison, final results, challenges and improvements | Chapters 5, 6 and 8 |

## Data and license

- **Source:** Kaggle, "Google Play Store Apps" by Lavanya Gupta (file `googleplaystore.csv`, 13 columns, collected in 2018).
- **License of the data:** CC BY-SA 4.0. If you share the cleaned data in this repository, keep the same license and the credit.
- **Citation:** L. Gupta, "Google Play Store Apps," Feb 2019. [Online]. Available: https://www.kaggle.com/lava18/google-play-store-apps

## Our decisions

| # | Decision | Why |
|---|---|---|
| 1 | Target: popular = 1,000,000 installs or more | Ratings are noisy for apps with few reviews and almost unrelated to the other columns. Installs are a clearer measure of success |
| 2 | Features: only things about the app itself | We want to know what about an app goes together with success |
| 3 | No Reviews and no Rating as features | Reviews appear after people install the app. With Reviews the test AUC would be 0.987 instead of 0.843 |
| 4 | Split first, fit every step on the training data only | To avoid data leakage |
| 5 | Main metric: F1 | A model that always says "not popular" is already right 64.8% of the time |

**Features:** category (one-hot, because the categories have no order), size, paid or free, price (log), content rating (an ordered number), days since the last update, minimum Android version. Empty values are filled with the median and marked with an extra column.

## Model comparison (5-fold cross-validation, training data)

| Model | Accuracy | Precision | Recall | F1 | AUC |
|---|---|---|---|---|---|
| Baseline | 0.648 | 0.000 | 0.000 | 0.000 | 0.500 |
| Logistic Regression | 0.756 | 0.716 | 0.511 | 0.596 | 0.810 |
| KNN | 0.749 | 0.654 | 0.610 | 0.631 | 0.797 |
| Decision Tree | 0.757 | 0.696 | 0.557 | 0.617 | 0.794 |
| **Random Forest** | 0.777 | 0.731 | 0.580 | 0.646 | 0.843 |

The Random Forest is the best on accuracy, precision, F1 and AUC. KNN has the best recall, and its F1 is close, so we chose the Random Forest mostly because of its AUC.

## What the model uses

Drop in AUC on the test set when one column is shuffled:

| Feature | Drop in AUC |
|---|---|
| Size (MB) | 0.096 |
| Days since the last update | 0.087 |
| Category | 0.076 |
| Price (log) | 0.014 |
| Minimum Android version | 0.014 |
| Paid or free | 0.013 |
| Content rating | 0.005 |

In the data, 56.7% of the apps updated in the last 30 days are popular, against 18.5% of the apps not updated for over a year, and 13.8% of the apps up to 5 MB are popular against 58.9% of the apps between 50 and 100 MB. These are links, not causes.

## Limits

- The data is from 2018 and installs are ranges (for example "1,000,000+"), so the one million line is approximate.
- We only have seven columns about the app. They say nothing about brand, marketing or quality, so the model misses about 4 popular apps in 10 (recall 0.594).
- The results are correlations. They do not show that changing one thing causes popularity.
- We used small grids for the model settings. The numbers can change a little with another version of scikit-learn.

## What we would do next

- Add features, for example words in the app name or the number of genres.
- Try gradient boosting, and choose the decision threshold with cross-validation to trade precision for recall.
- Predict the install tier instead of a yes or no.
- Use newer data.

## How to run it

1. Download `googleplaystore.csv` from the Kaggle page above and put it in your Google Drive (My Drive).
2. Open `AppCompass.ipynb` in Google Colab (use the badge at the top).
3. Click Runtime > Run all and allow access to Google Drive. It takes about 3 minutes.

The notebook uses `random_state=42` everywhere, and `kind='stable'` when it removes repeated apps, so the results are the same on every run. We checked that the results are the same with pandas 2.2 / scikit-learn 1.6 and with pandas 3.0 / scikit-learn 1.9.

## Team

[Team member names]

## License

Code: [choose a license, for example MIT]. Data: CC BY-SA 4.0 (see above).
