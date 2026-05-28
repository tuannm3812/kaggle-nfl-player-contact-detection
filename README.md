# NFL Player Contact Detection

![NFL football banner](https://cdn2.unrealengine.com/the-super-bowl-concludes-the-nfl-season-but-there-s-still-plenty-of-football-left-in-madden-24-1920x1080-5a2655f237f2.jpg)

![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white)
![Kaggle](https://img.shields.io/badge/Kaggle-NFL%20Player%20Contact%20Detection-20BEFF?style=flat-square&logo=kaggle&logoColor=white)
![Metric](https://img.shields.io/badge/Metric-MCC-2E7D32?style=flat-square)
![Status](https://img.shields.io/badge/Status-Notebook%207%20Challenger-2E7D32?style=flat-square)

## 1. Overview

This repository is a notebook-first workflow for Kaggle's
**1st and Future - Player Contact Detection** competition. The project uses NFL
tracking data, contact labels, baseline helmet boxes, video metadata, and game
video to detect when players experience external contact.

The broader goal is player safety: better contact detection can support injury
surveillance, workload analysis, and future prevention research.

## 2. Task and Goal

For each `contact_id`, predict whether contact occurred at a specific 10 Hz
timestep.

| Contact Type | Definition |
| --- | --- |
| Player-player | Two players are in contact. |
| Player-ground | One player has non-foot body contact with the ground. |

The submission target is binary:

```text
contact = 1 -> contact occurred
contact = 0 -> no contact occurred
```

## 3. Key Metric

The competition metric is **Matthews Correlation Coefficient (MCC)**. MCC is
the right metric because contact events are rare and accuracy would be
misleading.

Current EDA contact rates:

| Slice | Contact Rate |
| --- | ---: |
| Overall | 1.37% |
| Ground | 4.09% |
| Player-player | 1.11% |

Validation principles:

1. Split by `game_play` to avoid play-level leakage.
2. Tune thresholds on held-out plays.
3. Evaluate ground and player-player contact separately.
4. Confirm local gains with public and private leaderboard scores.

## 4. Progress

Notebook 5 is the current scored champion. Notebook 7 is the active local
challenger and should be submitted next.

| Rank | Notebook | Local MCC | Public MCC | Private MCC | Status |
| ---: | --- | ---: | ---: | ---: | --- |
| 1 | `5_type_specific_thresholds.ipynb` | 0.67650 | 0.65170 | 0.65127 | Current scored champion |
| 2 | `7_blended_type_models.ipynb` | 0.67944 | Pending | Pending | Submit challenger |
| 3 | `9_helmet_feature_model.ipynb` | Pending | Pending | Pending | New helmet-feature challenger |
| 4 | `6_type_specific_models.ipynb` | 0.67530 | 0.65212 | 0.64025 | Rejected: private regression |
| 5 | `4_nearest_player_and_smoothing.ipynb` | 0.67455 | 0.64497 | 0.64763 | Superseded |
| 6 | `3_tracking_feature_model.ipynb` | 0.65310 | 0.63075 | 0.62593 | Superseded |

Progress so far:

1. Tracking features produced the first competitive submission.
2. Nearest-player context and temporal smoothing improved generalization.
3. Separate ground/player-player thresholds improved player-player precision.
4. Fully separate type-specific models overfit the private split and were
   rejected.
5. Blended type-specific models improved local MCC and are the next submission
   candidate.
6. Helmet-box features are now packaged as the next offline-safe challenger.

## 5. Key Learnings

1. Player-player distance is a very strong signal; contact is rare beyond
   roughly `2` yards.
2. Ground contact needs different features from player-player contact,
   especially speed collapse, signed acceleration, and temporal windows.
3. Probability smoothing helps because contact labels and true contact moments
   can shift across neighboring 10 Hz steps.
4. Helmet-derived video features look promising: target visibility and
   helmet-pair pixel distance separate positive and negative rows.
5. Generic YOLOv8 worked for research-mode visual probing, but on the sample
   frame it was much sparser than the provided helmet boxes. Helmet-box
   features are the safer next production path.

## 6. Key Artifacts

| Artifact | Purpose |
| --- | --- |
| `notebooks/1_eda_contact_tracking_video_context.ipynb` | EDA, video overlay demo, tracking and helmet context. |
| `notebooks/5_type_specific_thresholds.ipynb` | Current scored champion. |
| `notebooks/7_blended_type_models.ipynb` | Current local challenger. |
| `notebooks/8_yolo_video_feature_probe.ipynb` | Research-mode video/YOLO probe. |
| `notebooks/9_helmet_feature_model.ipynb` | Helmet visibility, geometry, and pixel-distance feature challenger. |
| `docs/2_eda_insights.md` | EDA findings and feature ideas. |
| `docs/3_model_evaluation_progress.md` | Model scores, decisions, and next experiments. |

## 7. Business Applications

Reliable contact detection can turn raw tracking and video feeds into a more
complete view of player load and exposure. In practice, a system like this
could support:

1. **Player safety monitoring**: identify repeated contact events across plays,
   games, and seasons.
2. **Injury surveillance**: connect contact type, intensity, timing, and field
   context with injury review workflows.
3. **Coaching and technique review**: surface plays with high-contact patterns
   for teaching safer tackling, blocking, and landing mechanics.
4. **Medical and performance analytics**: quantify player contact burden beyond
   speed, acceleration, and distance traveled.
5. **Video review prioritization**: help analysts quickly locate moments where
   player-player or ground contact is likely.

The competition target is binary contact detection, but the real application is
decision support: making high-risk contact moments easier to measure, review,
and understand.

Tactical next experiments are tracked in
[`docs/3_model_evaluation_progress.md`](docs/3_model_evaluation_progress.md).
