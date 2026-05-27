# EDA Insights

## 1. Purpose

[`1_eda_contact_tracking_video_context.ipynb`](../notebooks/1_eda_contact_tracking_video_context.ipynb)
builds the evidence base for the NFL contact detection workflow. It now goes
beyond the starter notebook by separating player-player and player-ground
contact, analyzing temporal contact runs, profiling motion slices, and checking
helmet/video synchronization metadata safely.

## 2. Current Output Review

The latest executed EDA run loaded the core tables successfully:

| Dataset | Rows | Notes |
| --- | ---: | --- |
| `train_labels.csv` | 4,721,618 | Parsed into contact type and pair keys. |
| `sample_submission.csv` | 49,588 | Public mock test rows. |
| `train_player_tracking.csv` | 1,353,053 | 10 Hz tracking rows. |
| `test_player_tracking.csv` | 14,872 | Public mock test tracking rows. |
| `train_baseline_helmets.csv` | 3,783,616 | Baseline helmet boxes. |
| `test_baseline_helmets.csv` | 47,330 | Public mock test helmet boxes. |
| `train_video_metadata.csv` | 480 | Sideline and Endzone metadata rows. |
| `test_video_metadata.csv` | 4 | Public mock test metadata rows. |

The overall contact rate observed in the run was about `1.37%`. This confirms
that the task is severely imbalanced and that accuracy is not a useful
diagnostic. MCC, positive-rate control, recall, and slice analysis are more
important.

## 3. First Leaderboard Result

Notebook 3 produced the first scored submission:

| Submission | Public MCC | Private MCC | Local Validation MCC |
| --- | ---: | ---: | ---: |
| Tracking Feature, Version 3 | 0.63075 | 0.62593 | 0.65310 |

The public/private gap is small (`0.00482`), which is a good sign: the model is
not obviously overfit to the public split. The local validation score is about
`0.022` to `0.027` higher than leaderboard scores, so the grouped validation is
directionally useful but optimistic.

Important slice diagnostics from local validation:

| Contact Type | Local MCC | Actual Rate | Predicted Rate |
| --- | ---: | ---: | ---: |
| Ground | 0.41231 | 3.47% | 2.90% |
| Player-player | 0.71656 | 1.02% | 1.36% |

The model is strongest on player-player contact and materially weaker on
ground contact. This confirms that the next performance work should prioritize
ground-contact features and temporal smoothing rather than more static
player-player distance features.

Notebook 4 directly targets this gap with nearest-player density features and
play/pair probability smoothing. It should replace Notebook 3 only if grouped
validation beats `0.65310` or if the ground-contact slice improves without a
meaningful player-player regression.

Notebook 4 improved the first scored model:

| Submission | Public MCC | Private MCC | Local Validation MCC |
| --- | ---: | ---: | ---: |
| Nearest Player, Version 3 | 0.64497 | 0.64763 | 0.67455 |

The public/private scores rose by `+0.01422` and `+0.02170` over Notebook 3,
respectively. The private score is higher than public, which is a good sign for
generalization.

Notebook 5 is now the current scored champion:

| Submission | Public MCC | Private MCC | Local Validation MCC |
| --- | ---: | ---: | ---: |
| Type-Specific Thresh, Version 3 | 0.65170 | 0.65127 | 0.67650 |

The type-specific threshold gain is mostly player-player precision. The best
ground threshold stayed at `0.59`, while the best player-player threshold rose
to `0.70`. Ground MCC stayed at `0.50623`; player-player MCC improved from
`0.72226` to `0.72836`. The next refinement should therefore train separate
contact-type models or add dedicated ground-contact features.

Notebook 6 showed why the private leaderboard needs to override small public
gains. It raised public MCC from `0.65170` to `0.65212`, but private MCC fell
from `0.65127` to `0.64025`. Local validation also warned against it:
overall MCC fell to `0.67530`, and ground MCC dropped to `0.49675`. The next
refinement should use type-specific models only as a blended signal unless the
ground slice improves.

## 4. Fixed Error

The previous EDA notebook failed in the helmet/video metadata cell with:

```text
KeyError: "Column(s) ['video'] do not exist"
```

Root cause: `train_video_metadata.csv` contains timing columns such as
`start_time`, `end_time`, and `snap_time`, but it does not contain a `video`
filename column. Video filenames live in the baseline helmet files.

Fix: the updated notebook now summarizes video metadata by `dataset` and
`view`, and summarizes video filenames only from the helmet files.

## 5. Notebook Flow

| Step | Purpose |
| --- | --- |
| Setup and configuration | Resolve Kaggle paths, define runtime flags, frame-rate constants, and plotting defaults. |
| Load data | Read labels, submission, tracking, helmets, and train/test video metadata. |
| Data quality | Check missingness, duplicates, ID parsing, and train/test schema alignment. |
| Contact label balance | Measure class imbalance by player-player vs ground rows and by play. |
| Temporal dynamics | Analyze contact rate by step and contiguous positive-contact run duration. |
| Tracking context | Inspect field position, speed, distance, acceleration, orientation, and direction. |
| Ground-contact motion | Join ground labels to tracking motion fields to compare contact vs non-contact motion. |
| Player-player distance | Analyze distance distributions, contact rate by distance bin, and threshold behavior. |
| Helmet/video metadata | Safely summarize helmet box coverage, metadata duration, snap offset, and estimated frames. |
| Field visualization | Plot a selected play and step on a football field. |
| Distance baseline | Tune a distance threshold with play-grouped validation and MCC. |

## 6. Modeling Implications

The EDA supports a two-branch modeling plan:

- Player-player contact should start with distance, relative speed, relative
  acceleration, orientation alignment, nearest-player features, and temporal
  smoothing.
- Player-ground contact should be modeled separately because the distance
  baseline cannot detect it. Useful features should include speed, acceleration,
  signed acceleration, sudden motion changes, helmet box geometry, visibility,
  and temporal context.

Notebook 3 implements the first version of this plan using tracking-only
tabular features. It should be the next model to run when the distance baseline
underperforms because it can score both player-player rows and ground rows.

## 7. Validation Implications

Rows from the same `game_play` are highly correlated. Contact labels are also
temporally adjacent and can be noisy within roughly one 10 Hz timestep.

Default validation should therefore:

- group by `game_play`;
- tune thresholds on held-out plays;
- evaluate player-player and ground contact separately;
- inspect prediction smoothing after, not before, out-of-fold validation.

## 8. First Experiment

The first experiment remains a cleaned-up version of the starter notebook
baseline:

- parse `contact_id`;
- merge tracking coordinates for both players;
- compute Euclidean distance in yards;
- tune a hard distance threshold with MCC;
- write `submission.csv`.

Expected limitation: all ground rows are predicted as `0`, so recall on ground
contact will remain poor until a dedicated ground-contact branch is added.

## 9. Recommended Deep Dives

The next EDA passes should focus on the failure modes most likely to improve
MCC:

1. **Ground-contact motion windows**: compare speed, acceleration, signed
   acceleration, and distance traveled for `t-3` through `t+3` steps around
   ground-contact labels.
2. **Pair-distance dynamics**: analyze not just distance at step `t`, but
   distance change, closing speed, and minimum distance in a short window.
3. **Nearest-player context**: for each player-step, compute nearest teammate
   and opponent distances; contact risk depends on local density.
4. **Position and team slices**: evaluate contact rates and model errors by
   football position, same-team/opponent contact, and play duration.
5. **Helmet visibility**: summarize whether each player has Sideline/Endzone
   boxes near the synced frame, plus box size and movement. Missing or tiny
   boxes may explain video/model errors.
6. **Temporal smoothing sensitivity**: because labels can be off by about one
   10 Hz tick, validate whether short run filling or probability smoothing
   improves grouped out-of-fold MCC.

## 10. Improved Model Direction

[`3_tracking_feature_model.ipynb`](../notebooks/3_tracking_feature_model.ipynb)
adds an immediate model upgrade:

- keeps all positive labels and samples negatives for tractable training;
- validates on held-out plays with natural class balance;
- attaches player 1 tracking features for every row;
- attaches player 2 tracking features for player-player rows;
- creates prior-step motion deltas, cyclic direction/orientation encodings,
  pair distance, pair-distance change, relative motion, angular-difference,
  same-team, and ground-contact indicators;
- trains an offline-safe sklearn classifier;
- tunes the probability threshold directly for MCC;
- writes `submission.csv`.

This should be stronger than the distance baseline because it does not force all
ground-contact rows to zero.

## 11. Notebook Review

| Notebook | Current Role | Key Insight | Limitation |
| --- | --- | --- | --- |
| `1_eda_contact_tracking_video_context.ipynb` | Data understanding and failure-mode discovery. | The dataset is large, very imbalanced, temporally correlated, and contains distinct player-player and ground-contact problems. | It should be rerun in Kaggle after every major EDA addition because local execution cannot access competition data. |
| `2_distance_baseline_first_experiment.ipynb` | Starter-style sanity baseline. | Player-player distance is a useful lower bound and validates the submission path. | Ground rows are forced to `0`, so MCC is capped by missing ground-contact recall. |
| `3_tracking_feature_model.ipynb` | Current recommended model. | Tracking dynamics let the model learn both player-player and ground-contact patterns. | It is still tracking-only; helmet/video visibility and temporal smoothing remain the biggest likely next gains. |
| `4_nearest_player_and_smoothing.ipynb` | Superseded champion. | Local player density and temporal smoothing improved both public and private MCC. | Ground contact remains weaker than player-player contact. |
| `5_type_specific_thresholds.ipynb` | Current scored champion. | Separate thresholds improved player-player precision and lifted both public and private MCC. | Ground contact did not improve, so it is the next target. |
| `6_type_specific_models.ipynb` | Rejected challenger. | Separate models slightly raised public MCC. | Private MCC dropped sharply and ground local MCC regressed. |
| `7_blended_type_models.ipynb` | Current challenger. | Blends unified and type-specific probabilities to avoid full ground-model replacement. | Submit only if local MCC and ground-slice gates both clear Notebook 5. |

## 12. Path Decision

Earlier notebooks carried several `DATA_DIR` candidates while the Kaggle mount
was still unclear. The correct path is now confirmed as:

```text
/kaggle/input/competitions/nfl-player-contact-detection
```

All notebooks should use that single path. This keeps Kaggle failures easier to
debug and avoids accidentally reading from an older dataset-style mount.
