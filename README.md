# Does Success in Music Lead Artists to "Sell Out"?

A panel-data regression study of whether a song's popularity persists across an
artist's career, and whether changing your sound after a hit helps or hurts the
next release. Built in R, the project is framed as an **inference** problem
(understanding a mechanism) rather than a prediction problem, with a focus on
**regression to the mean**.

---

## TL;DR — Key Findings

- **Popularity is sticky, but mostly reverts.** A song's genre-adjusted popularity
  carries a coefficient of **β ≈ 0.316** onto the next song. That means roughly
  **68%** of any unexpected popularity ("excess" above the artist's genre baseline)
  regresses to the mean by the following release.
- **Sound alone explains almost nothing.** Adding six audio features of the previous
  song was jointly significant (F = 9.4, p < 0.001) but raised R² by only ~0.007 —
  a reminder that statistical significance at n ≈ 8,000 is not the same as a
  meaningful effect.
- **Changing your sound did *not* hurt — if anything, it helped.** The interaction
  between previous popularity and the *magnitude* of sound change was positive and
  significant (δ ≈ 0.162, p < 0.001). Artists who changed their sound more after a
  hit retained slightly more popularity, the opposite of what a "sell out and repeat
  the formula" strategy predicts.
- **Direction didn't matter.** Moving *toward* vs. *away from* the genre mainstream
  showed no significant effect (δ ≈ −0.013, CI straddles zero).
- **All models together explain only ~11% of variance**, consistent with prior work
  showing music success is driven heavily by social/timing factors outside audio
  features.

---

## Research Questions

1. **Persistence.** If a song outperforms its genre average, does the artist's *next*
   song tend to do the same?
2. **Sound & strategy.** Does the previous song's sound — and the decision to change
   it — predict the popularity of the next song? In other words, does "selling out"
   actually work commercially?

---

## Data

| | |
|---|---|
| Source | "Spotify Songs" dataset, ~32,833 tracks pulled from the Spotify API (late 2019) |
| Raw observations | 32,833 tracks |
| After de-duplicating on `track_id` | 28,352 unique songs / 10,692 artists |
| Analysis panel (artists with ≥ 5 songs, valid dates) | 13,372 tracks / 1,240 artists |
| Final unit of analysis | **8,009 cross-album consecutive song pairs** from 1,202 artists |
| Genres | EDM, Latin, Pop, R&B, Rap, Rock |

> **Note on the data source:** `spotify_songs.csv` is a third-party dataset, not
> original to this project. If you know the original source (e.g. a Kaggle page or
> the TidyTuesday 2020-01-21 release), add the link here so others can trace
> provenance and check licensing terms.

The unit of observation is a **consecutive pair of songs by the same artist, across
album cycles** — i.e. `(song n−1, song n)`. Intra-album pairs are dropped because
tracks on one album share a release date, making within-album ordering arbitrary.

---

## Method

Three nested OLS models of increasing complexity, all with **artist-clustered,
heteroskedasticity-robust (sandwich) standard errors** (`sandwich::vcovCL`,
`cluster = ~track_artist`). Because the sample is large enough that F-tests flag
almost any non-zero coefficient as significant, the project deliberately reports
**ΔR² and effect sizes alongside p-values** to judge whether added complexity is
*meaningful*, not just *significant*.

Two engineered variables anchor the analysis:

- **`pop_resid`** — popularity with genre fixed effects removed (residual from
  regressing `track_popularity` on `playlist_genre`), so only *within-genre* signal
  is studied.
- **`sound_change`** — Euclidean distance in (danceability, energy, valence) space
  between consecutive songs; plus a *directional* version (`centroid_approach`) built
  from leave-one-artist-out genre centroids, and a 9-feature z-scored robustness
  version.

| Model | Specification | Question |
|---|---|---|
| **1** | `pop_resid_n ~ pop_resid_{n−1}` | Does popularity persist? |
| **2** | Model 1 + 6 audio features of song n−1 | Does the previous sound matter? |
| **3** | Model 2 + `pop_resid_{n−1} × sound_change` | Does changing your sound after a hit help? |

Robustness: Model 3 is re-fit per genre, with a directional (centroid) sound-change
measure, and with a 9-feature z-scored measure.

---

## Results at a glance

| Model | Key estimate | SE | R² | Notes |
|---|---|---|---|---|
| 1 | β (persistence) = 0.316 | 0.015 | 0.097 | 95% CI (0.287, 0.346), excludes 0 and 1 |
| 2 | audio features jointly | — | 0.104 | F = 9.4, p < 0.001, ΔR² ≈ 0.007 |
| 3 | δ (interaction) = 0.162 | 0.077 | 0.109 | F = 16.1, p < 0.001, CI (0.012, 0.313) |
| 3-dir | δ (centroid) = −0.013 | 0.073 | 0.104 | not significant |
| 3-alt | δ (9-feature, z) = 0.030 | 0.009 | 0.110 | significant, same sign |

---

## Repository contents

| File | Description |
|---|---|
| `spotify_regression.Rmd` | Full reproducible analysis source (data cleaning, variable construction, all three models, figures) |
| `spotify_regression.pdf` | Knitted report with narrative, figures, and tables |
| `spotify_songs.csv` | Input dataset (~7.7 MB) |
| `README.md` | This file |
| `LICENSE` | MIT license (covers the code/analysis) |

---

## Reproducing the analysis

Requires **R** (≥ 4.0) and a LaTeX engine (e.g. TinyTeX) for the PDF.

```r
install.packages(c(
  "tidyverse", "sandwich", "lmtest",
  "corrplot", "knitr", "kableExtra", "gridExtra"
))
```

Then, from this directory, knit the report:

```r
rmarkdown::render("spotify_regression.Rmd")
```

The `read_csv("spotify_songs.csv")` call assumes the CSV sits in the same folder as
the `.Rmd`. If you move the data into a `data/` subfolder, update that path.

---

## Limitations

- **Omitted-variable bias.** Label investment likely rises after a hit *and* drives
  popularity, biasing the persistence coefficient upward; it may also be correlated
  with the sound-change measure.
- **Sampling.** The data is drawn from editorial playlists, which over-represent
  commercially successful artists.
- **Low explained variance (~11%).** Audio features and prior popularity leave most
  of the variation unexplained — consistent with research finding that music success
  is largely social and timing-driven (Salganik et al., 2006; Askin & Mauskapf, 2017).

---

## References

- Askin, N., & Mauskapf, M. (2017). What makes popular culture popular? *American Sociological Review, 82*(5), 910–944.
- Galton, F. (1886). Regression towards mediocrity in hereditary stature. *Journal of the Anthropological Institute, 15*, 246–263.
- Salganik, M. J., Dodds, P. S., & Watts, D. J. (2006). Experimental study of inequality and unpredictability in an artificial cultural market. *Science, 311*(5762), 854–856.

---

*Author: Josh Nguyen*
