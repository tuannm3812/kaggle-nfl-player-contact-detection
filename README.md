# NFL Player Contact Detection

![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white)
![Kaggle](https://img.shields.io/badge/Kaggle-NFL%20Player%20Contact%20Detection-20BEFF?style=flat-square&logo=kaggle&logoColor=white)
![Metric](https://img.shields.io/badge/Metric-MCC-2E7D32?style=flat-square)
![Status](https://img.shields.io/badge/Status-Tracking%20Model%20Ready-2E7D32?style=flat-square)

This repository contains a notebook-first workflow for Kaggle's
**NFL Player Contact Detection** competition. The task is to identify moments
when a player is in contact with another player or with the ground by combining
Next Gen Stats tracking data, baseline helmet detections, video metadata, and
contact labels.

Expected Kaggle input path:

```text
/kaggle/input/competitions/nfl-player-contact-detection
```

Input files:

- `train_labels.csv`: contact labels for player-player and player-ground pairs.
- `train_player_tracking.csv`: 10 Hz training tracking data.
- `test_player_tracking.csv`: 10 Hz test tracking data.
- `train_baseline_helmets.csv`: baseline helmet boxes for training videos.
- `test_baseline_helmets.csv`: baseline helmet boxes for test videos.
- `train_video_metadata.csv`: video timing metadata.
- `sample_submission.csv`: exhaustive test `contact_id` rows.

## Repository Structure

```text
.
|-- README.md
|-- docs/
|   |-- coding_standards.md
|   |-- 1_instructions.md
|   `-- 2_eda_insights.md
`-- notebooks/
    |-- 1_eda_contact_tracking_video_context.ipynb
    |-- 2_distance_baseline_first_experiment.ipynb
    `-- 3_tracking_feature_model.ipynb
```

## Notebook Workflow

| Notebook | Purpose |
| --- | --- |
| `1_eda_contact_tracking_video_context.ipynb` | Data quality, contact label balance, tracking context, helmet/video metadata checks, and distance-baseline validation. |
| `2_distance_baseline_first_experiment.ipynb` | Reusable first experiment based on the starter notebook: tune a player-player distance threshold with MCC and write `submission.csv`. |
| `3_tracking_feature_model.ipynb` | Offline-safe tracking-feature classifier for player-player and ground rows, grouped validation, MCC threshold tuning, and `submission.csv`. |

## Current Modeling Direction

The first experiment uses only player tracking distance. It is deliberately
simple and submission-safe, but it predicts no ground contact.

1. Parse `contact_id` into `game_play`, `step`, `nfl_player_id_1`, and
   `nfl_player_id_2`.
2. Merge tracking coordinates for both players.
3. Predict player-player contact when Euclidean distance is below a tuned yard
   threshold.
4. Predict no ground contact in this baseline, then improve that branch in the
   next experiment using body/helmet/video features.

Notebook 3 is the current recommended modeling path. It trains a sampled
tracking-feature classifier, adds prior-step movement and pair-distance-change
features, validates on held-out plays with natural class balance, tunes a hard
threshold for MCC, and writes `submission.csv`.

The competition metric is Matthews Correlation Coefficient, so notebooks should
report MCC for hard predictions instead of optimizing accuracy.
