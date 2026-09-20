# Spotify Track Popularity Prediction

**HarvardX PH125.9x Data Science Capstone** · R · January 2021

Predicting how popular a Spotify track is (a 0-100 score) from its energy level and release year, using five models of increasing complexity. The final random forest cuts prediction error (RMSE) from about **21.8 to about 10.8**, a reduction of more than 50% versus a predict-the-average baseline.

Along the way, the analysis surfaced a **sampling bias in the dataset itself**, which shaped how the results should be interpreted (see [Key finding](#key-finding-the-data-is-not-a-random-sample)).

## Question

Can a track's popularity be predicted from a small number of audio and track features, and how much does each modeling step improve on a naïve baseline?

## Data

- **Source:** Spotify 1921-2020 dataset on Kaggle: 160,000+ tracks released between 1921 and 2020, covering genres from classical and jazz to metal and techno.
- **Target:** `popularity`, a 0-100 score Spotify calculates from a track's total plays and how recent those plays are.
- **Predictors used:** `energy` (0.0-1.0) and release `year`.
- **Not used:** artist, track name/ID, and release date (dropped in preprocessing), along with the remaining audio features (acousticness, danceability, loudness, valence, etc.).
- **Split:** 90% train / 10% test with `set.seed(1)`. Test rows were filtered so every energy value and year also appears in the training set.

## Approach

Each model was evaluated on the held-out test set using the root mean squared error (RMSE).

| # | Model | Change vs. previous model (RMSE) |
|---|-------|----------------------------------|
| 1 | Baseline: predict the global mean popularity | RMSE ≈ 21.8 |
| 2 | Energy effect | ~13% lower |
| 3 | Energy + year effect | ~28% lower |
| 4 | Regularized energy + year effect (λ = 10) | ~4.5% lower |
| 5 | Random forest on energy + year | ~17% lower, **final RMSE ≈ 10.79** |

Regularization penalizes effect estimates built on very few observations (some energy levels and years have only a handful of tracks). The random forest was tuned with 5-fold cross-validation over `mtry`.

## Key finding: the data is not a random sample

Exploratory analysis showed that the number of tracks per year plateaus at about 2,000 for years after roughly 1950. That pattern is unlikely to occur naturally, and suggests the dataset contains only the top ~2,000 tracks per year for those years. This creates a built-in popularity bias: tracks released after 1950 have much higher average popularity than earlier ones, and release year correlates strongly with popularity (r ≈ 0.86). The model's accuracy therefore reflects, in part, how the dataset was assembled and not only how music behaves.

Other observations from exploration:

- Popularity is heavily skewed: most tracks score near zero, and only about 1% score 80 or above.
- Average popularity rises with energy, but energy levels with few tracks produce noisy averages, which is the motivation for regularization.

## Limitations and known issues

- **Only two predictors.** Many available features (acousticness, danceability, loudness, valence) were not modeled. Adding them is the most obvious way to improve accuracy.
- **λ was tuned on the test set.** The regularization parameter was chosen by minimizing the RMSE on the held-out test set, so Model 4's reported RMSE is slightly optimistic. A cleaner approach is to select λ with cross-validation on the training data only. (The random forest's `mtry` was tuned with cross-validation on the training set, so the final RMSE is not affected by this.)
- **Small forest.** The random forest uses only 2 trees to keep training time low; more trees would likely improve stability.
- **Sampling bias.** As described above, the dataset appears to over-represent popular tracks after 1950, which limits how far the model generalizes.
- **No user-level data.** The dataset has no listening-behavior data, so it can't support a recommendation system.

## Repository contents

| File | Description |
|------|-------------|
| `Spotify Popularity Prediction Project.Rmd` | Full analysis: exploration, models, results |
| `Spotify Popularity Prediction Project.pdf` | Rendered report |
| `Spotify Popularity Prediction Project R Script.R` | Standalone R script |

## Running it

Requires R with the `tidyverse`, `caret`, `data.table`, and `ggcorrplot` packages (the script installs any that are missing). The script downloads the dataset automatically. Knit the `.Rmd` or run the `.R` script; the random forest step takes a few minutes.

## Tools

R · tidyverse · caret · ggplot2 · R Markdown

## Author

Jeffrey Chin
