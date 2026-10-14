# Squirrel Behaviour vs Time of Day

Can the behaviour of Central Park's grey squirrels tell you whether it's morning or afternoon?

This project uses the [2018 Central Park Squirrel Census](https://data.cityofnewyork.us/Environment/2018-Central-Park-Squirrel-Census-Squirrel-Data/vfnx-vebw) to ask:

> Can grey squirrel behaviour and interaction patterns predict the time of day (AM vs PM), and what do these patterns suggest about the impact of human activity on squirrels across different times?

It started as a group project for COMP20008 (Elements of Data Processing) at the University of Melbourne.

## Approach

1. **Feature engineering.** The 13 boolean behaviour columns (running, eating, kuks, tail flags, approaches, …) are grouped into six scores and normalised to [0, 1]:
   *Movement, Feeding, Vocalisation, Communication, Human interaction, Degree of friendliness.*
2. **Correlation analysis.** Pearson correlation and mutual information between each score and the shift (AM/PM).
3. **Supervised learning.** k-nearest neighbours and a decision tree, each tuned with 5-fold stratified cross-validation and compared against a random baseline.
4. **Clustering.** K-means (elbow method) and agglomerative (Ward) clustering, run separately on AM and PM sightings and plotted over a map of the park.

## Results

Test set: 495 sightings (215 AM, 280 PM).

| Model | Accuracy | Weighted F1 |
|---|---|---|
| Random baseline (uniform) | 0.52 | 0.52 |
| KNN (k = 35, distance-weighted) | 0.56 | 0.56 |
| Decision tree (max depth = 9) | **0.59** | **0.57** |

- **Behaviour is only weakly linked to time of day.** All Pearson correlations are |r| < 0.1. *Feeding* is slightly higher in the PM (r = 0.08) and *Human interaction* slightly lower (r = −0.10). Mutual information is close to zero for every feature.
- **The models beat random guessing, but only barely beat always guessing "PM".** Always guessing PM would score 0.57 on this test set.
- **Clustering finds activity types, not time-of-day groups.** Both shifts split into similar *feeder*, *mover* and *human-interaction* clusters.

**Conclusion:** squirrel behaviour (as recorded in the census) is not a reliable signal for time of day.

## Repository structure

```
├── data/
│   ├── squirrel.csv     # one row per squirrel sighting (main dataset)
│   ├── hectare.csv      # per-hectare survey conditions
│   └── stories.csv      # free-text squirrel & park stories
├── notebooks/
│   └── squirrel_analysis.ipynb
├── assets/              # optional map background (see below)
├── requirements.txt
└── LICENSE
```

## How to run

You need Python 3.11 or newer.

```bash
git clone <this-repo-url>
cd squirrel-behaviour
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook notebooks/squirrel_analysis.ipynb
```

Run the notebook from the `notebooks/` folder, because data paths are relative to it (`../data/`).

**Map background (optional).** The cluster scatter plots can draw over a satellite image of Central Park at `assets/centralpark.png`. The image comes from a third-party map and is not included in this repo. If the file is missing, the plots still render, just without the background.

**Reproducibility.** Every split, fold and model uses `random_state=42`. The features are discrete, so many sightings share identical feature vectors. KNN tie-breaking can therefore differ slightly between platforms. The saved outputs were produced on Google Colab.

## Limitations and future work

- The classes are imbalanced (≈57% PM), so accuracy flatters the models. Balanced accuracy, or a majority-class baseline, would be a fairer comparison.
- The decision tree was tuned on normalised features with a stratified split, but the final tree was fitted on raw features with a non-stratified split. Aligning the two would make the comparison cleaner.
- The bag-of-words features from `stories.csv` are computed but not yet used in any model.
- The census covers only two weeks in October 2018, so seasonal effects can't be studied.

## Data source and license

The data comes from [NYC Open Data](https://opendata.cityofnewyork.us/) (2018 Central Park Squirrel Census: [Squirrel Data](https://data.cityofnewyork.us/Environment/2018-Central-Park-Squirrel-Census-Squirrel-Data/vfnx-vebw), [Hectare Data](https://data.cityofnewyork.us/Environment/2018-Central-Park-Squirrel-Census-Hectare-Data/ej9h-v6g2), [Stories](https://data.cityofnewyork.us/Environment/2018-Central-Park-Squirrel-Census-Stories/gfqj-f768)). Use of the data is governed by the [NYC Open Data Terms of Use](https://opendata.cityofnewyork.us/overview/#termsofuse).

The code is released under the [MIT License](LICENSE).

## Authors

Brian Nguyen, Minh Hoang Ha, Tony Tran, Faiyaz Rashid
# Squirrel-Behaviour
