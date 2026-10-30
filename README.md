# Beer Rating and Review

A friend is about to open a beer store and has no idea what to put on the shelves. All they have is a dataset of beer ratings and reviews. This project is the analysis that answers: **which beers should the store stock, and why?**

The whole analysis is in [`exploratory_analyses/eda.ipynb`](exploratory_analyses/eda.ipynb). It is the first cycle of the CRISP-DS methodology.

> **Important:** the data comes from a **very specific community of craft beer lovers**. Everything here is what *they* like, and it may not be what the customers of the store like.

## The data

`data/train.csv` has 37,500 reviews of 1,731 beers, written by 7,441 users. Each row is one review and has three kinds of information:

- **Beer:** ABV, beer id, brewery id, name and style.
- **Review:** ratings (appearance, aroma, palate, taste and overall, from 1.0 to 5.0), text and date.
- **User:** profile name, age, birthday and gender (most of them are empty).

## Questions and answers

### 1. First impressions

- The dataset is a catalog of beers, with reviews, written by users.
- It is **not a sample of the whole beer market**: the most reviewed styles are strong, dark or hoppy beers (Imperial Stout, IPA, Double IPA, Scotch Ale), and the median ABV is 6.9%, high for a beer.
- The 95 styles are very unbalanced: the top 15 have 69% of the reviews, and 43 styles have less than 100 reviews.
- Age, birthday and gender are almost empty (about 80% and 60% of nulls).

### 2. Can we trust it?

Mostly yes, with some care. The ratings, the beer information and the timestamps are consistent.

| Issue | Size | Decision |
|---|---|---|
| Nulls in age, birthday and gender | ~79% (age, birthday), ~60% (gender) | Do not use age, birthday and gender |
| Default birthdays (for example Jan 1, 1900) | 65 reviews, 4 users | One more reason to not use age |
| Rating 0.0 outside the 1.0-5.0 scale | 1 review | Drop the row |
| Repeated review texts | 16 reviews | Keep, drop only if we use the text |
| No `profileName` | 5 reviews | `profileName` works as the user key, ignore those 5 in user analysis |
| Same beer name with different `beerId` | 27 names, 1,175 reviews | Always use `beerId`, not the name |
| Extreme ABV values | 5 reviews above 20%, 4 at 1% or less | Real values, keep them but treat as outliers in the model |

All the reviews are kept, including repeated reviews of the same beer by the same user, because they are useful to evaluate the quality of a beer over time.

### 3. What drives a good rating?

- A linear regression with the four sub-ratings explains 65% of the variance of the overall rating.
- **Taste is by far the most important** (standardized coefficient 0.55), then palate (0.24). Aroma (0.06) and appearance (0.04) matter little. Taste alone explains 62%.
- Before tasting, the **style is a good clue**: among the 52 styles with 100 or more reviews, the mean goes from 2.76 (Light Lager) to 4.33 (American Double / Imperial Stout).
- These are what the users value, not a cause. The sub-ratings are not known before the review, so they cannot predict a new beer.

### 4. The store decision

The unit is the beer (`beerId`). Beers with few reviews are not trusted, so the ranking uses a **Bayesian average** (weight 30), and the beers with 10 or more reviews (349) are grouped by quality and popularity:

| Group | Rule | Beers |
|---|---|---|
| Star | score of 4.0 or more and 100 or more reviews | 24 |
| Potential | score of 4.0 or more and less than 100 reviews | 26 |
| Popular, above average | 100 or more reviews and score between the global mean and 4.0 | 8 |
| Popular, below average | 100 or more reviews and score below the global mean | 32 |

Half of the Stars are from Founders, so a shelf made only by the ranking would be concentrated in one brewery and in a few strong styles.

As a **starting point** (a judgment, not a result of the data, because there are no sales), a catalog to please different kinds of customers could be:

- **~50% Stars:** the base of the shelf.
- **~25% Potential:** well rated but still little known, to make the store different.
- **~20% Popular above average:** a solid second line.
- **~5% Popular below average:** a small share, for the ones just below the average, which can change group with a few new reviews.

These shares should be adjusted with the real sales of the first months.

### 5. Looking ahead

A new beer has no reviews, so the simplest prediction is the **mean rating of its style**. Validated by `beerId` (5 groups, all the reviews of a beer stay together), the error (RMSE) goes from 0.70 (global mean) to 0.66 (style mean). It is a small gain: the style explains only part of the rating. When the beer receives its first reviews, the prediction should be replaced by the Bayesian average of the beer.

### 6. The 2-minute version

- **People reward the taste.** Before tasting, the style is a good clue, but it is only a clue.
- Everything here is the opinion of a **very specific community of craft beer lovers**. For example, light lagers have low ratings here, but they can sell well.
- Do not focus only on the Stars. A good catalog has to please different kinds of customers.
- The dataset has no price, sales or costs. The recommendation is about perceived quality, and it needs to be checked with local demand before the final decision.

## Next steps (TO DO in the notebook)

- ABV x rating, also inside each style.
- Brewery x rating, with a minimum number of reviews.
- Joint model with style, ABV and brewery.
- Machine learning model for a new beer (Ridge and gradient boosting), to beat the style mean, with validation by `beerId`, in time and by brewery.

## How to run

The project uses Python 3.12 and [uv](https://docs.astral.sh/uv/).

```bash
uv sync
```

Then open `exploratory_analyses/eda.ipynb` and run the cells (the `ipykernel` is in the dev group).

If the network uses a TLS-intercepting proxy, `native-tls = true` is already set in `pyproject.toml`.

## Project structure

```
data/                      beer reviews dataset (train.csv)
exploratory_analyses/      eda.ipynb, the full analysis
pyproject.toml             dependencies (pandas, matplotlib, statsmodels, scikit-learn)
```
