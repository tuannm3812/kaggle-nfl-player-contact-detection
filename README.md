# NFL Player Contact Detection

![NFL football banner](https://cdn2.unrealengine.com/the-super-bowl-concludes-the-nfl-season-but-there-s-still-plenty-of-football-left-in-madden-24-1920x1080-5a2655f237f2.jpg)

![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white)
![Kaggle](https://img.shields.io/badge/Kaggle-NFL%20Player%20Contact%20Detection-20BEFF?style=flat-square&logo=kaggle&logoColor=white)
![Metric](https://img.shields.io/badge/Metric-MCC-2E7D32?style=flat-square)
![Status](https://img.shields.io/badge/Status-Notebook%205%20Champion-2E7D32?style=flat-square)

This repository contains a notebook-first workflow for Kaggle's
**NFL Player Contact Detection** competition. The goal is to detect external
contact between NFL players, and between players and the ground, using tracking
data, baseline helmet detections, video metadata, and contact labels.

The current best submission is a tracking-only model with nearest-player
context, temporal smoothing, and separate ground/player-player thresholds.

## Current Best Result

| Rank | Notebook | Submission | Local MCC | Public MCC | Private MCC | Status |
| ---: | --- | --- | ---: | ---: | ---: | --- |
| 1 | `5_type_specific_thresholds.ipynb` | Type-Specific Thresh, Version 3 | 0.67650 | 0.65170 | 0.65127 | Current champion |
| 2 | `6_type_specific_models.ipynb` | Type-specific models | Pending | Pending | Pending | Next challenger |
| 3 | `4_nearest_player_and_smoothing.ipynb` | Nearest Player, Version 3 | 0.67455 | 0.64497 | 0.64763 | Superseded |
| 4 | `3_tracking_feature_model.ipynb` | Tracking Feature, Version 3 | 0.65310 | 0.63075 | 0.62593 | Superseded |

Notebook 5 improved over Notebook 4 by `+0.00673` public MCC and `+0.00364`
private MCC. The gain came mainly from improving player-player precision:
the best ground threshold stayed at `0.59`, while the best player-player
threshold moved to `0.70`.

## Key Takeaways

- MCC is the right decision metric. Accuracy is misleading because contact
  events are rare.
- Grouped validation by `game_play` is essential. Row-level random splits leak
  temporal and play context.
- Tracking data alone is already strong enough for a competitive baseline.
- Ground contact and player-player contact behave differently and should be
  evaluated separately.
- Temporal smoothing helps because labels and true contact events can shift
  across neighboring 10 Hz steps.
- The remaining weakness is ground-contact detection. Notebook 6 tests whether
  separate ground/player-player models can improve that slice.

## Lessons Learned

- Start with a submission-safe baseline before adding richer features.
- Keep notebook versions explicit. Each Kaggle run should print
  `NOTEBOOK_VERSION` before any expensive work starts.
- Preserve useful Kaggle outputs when the code has not changed; clear outputs
  only when they are stale or failed.
- Track public and private leaderboard scores beside local validation. Local
  MCC ranks experiments, while leaderboard scores confirm generalization.
- Small threshold changes can outperform larger model changes when the model
  is already well calibrated enough for ranking.

## Data Path

All notebooks use the fixed Kaggle competition path:

```text
/kaggle/input/competitions/nfl-player-contact-detection
```

Required files:

- `train_labels.csv`
- `sample_submission.csv`
- `train_player_tracking.csv`
- `test_player_tracking.csv`
- `train_baseline_helmets.csv`
- `test_baseline_helmets.csv`
- `train_video_metadata.csv`
- `test_video_metadata.csv`

## Repository Structure

```text
.
|-- README.md
|-- docs/
|   |-- coding_standards.md
|   |-- 1_instructions.md
|   |-- 2_eda_insights.md
|   `-- 3_model_evaluation_progress.md
`-- notebooks/
    |-- 1_eda_contact_tracking_video_context.ipynb
    |-- 2_distance_baseline_first_experiment.ipynb
    |-- 3_tracking_feature_model.ipynb
    |-- 4_nearest_player_and_smoothing.ipynb
    |-- 5_type_specific_thresholds.ipynb
    `-- 6_type_specific_models.ipynb
```

## Notebook Workflow

| Notebook | Purpose |
| --- | --- |
| `1_eda_contact_tracking_video_context.ipynb` | Data quality, contact balance, tracking context, helmet/video metadata checks, and distance-baseline validation. |
| `2_distance_baseline_first_experiment.ipynb` | Starter-style distance baseline with grouped validation and `submission.csv` generation. |
| `3_tracking_feature_model.ipynb` | Tracking-feature classifier for player-player and ground rows with MCC threshold tuning. |
| `4_nearest_player_and_smoothing.ipynb` | Adds nearest-player density features and play/pair probability smoothing. |
| `5_type_specific_thresholds.ipynb` | Tunes separate ground and player-player thresholds after smoothing. |
| `6_type_specific_models.ipynb` | Trains separate ground and player-player models before smoothing and threshold tuning. |

## Modeling Direction

The project has moved from a simple distance threshold to a stronger
tracking-feature pipeline:

1. Parse `contact_id` into play, step, and player identifiers.
2. Attach tracking features for player 1 and player 2.
3. Add pairwise distance, relative motion, nearest-player context, and
   prior-step dynamics.
4. Validate on held-out plays with natural class balance.
5. Tune hard thresholds with Matthews Correlation Coefficient.
6. Smooth probabilities within each play/contact-pair sequence.
7. Evaluate ground and player-player slices separately before submitting.

Next priority: run Notebook 6 and submit only if grouped validation beats
Notebook 5's `0.67650` local MCC, with no meaningful regression in the
ground-contact slice.
