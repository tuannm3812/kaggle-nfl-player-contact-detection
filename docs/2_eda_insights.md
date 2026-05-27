# EDA Insights

## 1. Purpose

[`1_eda_contact_tracking_video_context.ipynb`](../notebooks/1_eda_contact_tracking_video_context.ipynb)
builds the evidence base for the NFL contact detection workflow. It now goes
beyond the starter notebook by separating player-player and player-ground
contact, analyzing temporal contact runs, profiling motion slices, and checking
helmet/video synchronization metadata safely.

The EDA notebook should stay focused on understanding the game context and
failure modes. Model-score tracking belongs in
[`3_model_evaluation_progress.md`](3_model_evaluation_progress.md); this file
summarizes what the data tells us, which visual checks matter, and which
deep-dives should guide the next notebooks.

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
| Video/helmet overlay demo | Map a labeled step to a video frame, overlay baseline helmet boxes, and inspect whether target players are visible. |
| Distance baseline | Tune a distance threshold with play-grouped validation and MCC. |

## 6. EDA Deep-Dive Themes

The strongest EDA direction now is not simply more summary tables. We should
use EDA to understand moments of contact as short football sequences:

1. **Field simulation view**: animate or step through tracking positions for a
   selected play, coloring active contact pairs and ground-contact players.
   This makes temporal label noise visible and helps explain why smoothing
   improves MCC.
2. **Video frame demo**: show Sideline and Endzone frames near a positive
   contact step with baseline helmet boxes overlaid. This validates the
   `step -> frame` mapping and reveals occlusion or missing-player cases.
3. **Ground-contact windows**: compare speed, acceleration, signed
   acceleration, orientation change, and distance traveled from `t-3` through
   `t+3`. Ground contact is still the weakest slice.
4. **Pair-distance dynamics**: analyze distance, distance change, closing
   speed, and minimum distance inside short windows. A single-distance cutoff
   misses late-arriving or crowded contacts.
5. **Helmet visibility and geometry**: summarize target-player box visibility,
   helmet box area, box movement, and pixel distance in each view. These
   features are cheap video proxies and should come before a heavy CNN.
6. **Position/team context**: split contact rates by position, same-team versus
   opponent pairs, and local player density. The leaderboard writeups confirm
   same-team/contact-context features can matter.

## 7. Video and YOLO Direction

The new
[`8_yolo_video_feature_probe.ipynb`](../notebooks/8_yolo_video_feature_probe.ipynb)
is an investigation notebook, not a champion replacement. It tests three
questions:

1. Can we reliably map a 10 Hz label step to the correct Sideline/Endzone video
   frame?
2. Do the provided helmet boxes expose useful visibility and box-geometry
   features?
3. If offline YOLO weights are attached, do YOLO detections add useful body
   context beyond the helmet boxes?

The conservative path is to first add helmet-derived features to the current
tracking pipeline. YOLO should only become a modeling dependency if the probe
shows signal that is not already captured by the baseline helmet detections.

## 8. Lessons from Team Hydrogen

The 2nd-place Team Hydrogen writeup is useful because it explains why video
worked for top solutions. Their biggest ideas were:

1. **Validation first**: they used Stratified Group KFold by `game_key`. This
   confirms our grouped-validation habit is directionally correct, but our
   single split should eventually become multi-fold if runtime allows.
2. **Temporal video crops**: their CNN looked at multiple frames before and
   after the target step. This matches our smoothing results: contact is a
   short event, not a single-row phenomenon.
3. **Early fusion of Sideline and Endzone**: they concatenated both views so
   the model could learn from complementary camera angles.
4. **Tracking encoded into image channels**: they did not rely on raw pixels
   alone. Distance, same-team flags, and movement features were encoded into
   CNN input channels.
5. **Interpolation**: they linearly interpolated tracking and helmet boxes from
   10 Hz/available detections toward video-frame timing. This is likely more
   important than adding a generic detector immediately.
6. **Stage 2 blending**: their final result blended CNN predictions, smoothed
   predictions, and a small LGBM stage-2 model with selected tracking features.

Practical lesson for this repo: we should not jump straight from tracking-only
models to a full EfficientNet video ensemble. The next realistic steps are:

1. add helmet visibility and box-distance features;
2. add short-window tracking features and interpolation;
3. test a small video/YOLO probe on sampled frames;
4. blend video-derived probabilities with the existing tracking champion only
   after local grouped validation improves.

## 9. Modeling Implications

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

## 10. Validation Implications

Rows from the same `game_play` are highly correlated. Contact labels are also
temporally adjacent and can be noisy within roughly one 10 Hz timestep.

Default validation should therefore:

- group by `game_play`;
- tune thresholds on held-out plays;
- evaluate player-player and ground contact separately;
- inspect prediction smoothing after, not before, out-of-fold validation.

## 11. First Experiment

The first experiment remains a cleaned-up version of the starter notebook
baseline:

- parse `contact_id`;
- merge tracking coordinates for both players;
- compute Euclidean distance in yards;
- tune a hard distance threshold with MCC;
- write `submission.csv`.

Expected limitation: all ground rows are predicted as `0`, so recall on ground
contact will remain poor until a dedicated ground-contact branch is added.

## 12. Recommended Deep Dives

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

## 13. Improved Model Direction

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

## 14. Notebook Review

| Notebook | Current Role | Key Insight | Limitation |
| --- | --- | --- | --- |
| `1_eda_contact_tracking_video_context.ipynb` | Data understanding, failure-mode discovery, and visual context. | The dataset is large, very imbalanced, temporally correlated, and contains distinct player-player and ground-contact problems. The new video overlay demo checks whether target players are visible at the synced frame. | It should be rerun in Kaggle after every major EDA addition because local execution cannot access competition data. |
| `2_distance_baseline_first_experiment.ipynb` | Starter-style sanity baseline. | Player-player distance is a useful lower bound and validates the submission path. | Ground rows are forced to `0`, so MCC is capped by missing ground-contact recall. |
| `3_tracking_feature_model.ipynb` | Current recommended model. | Tracking dynamics let the model learn both player-player and ground-contact patterns. | It is still tracking-only; helmet/video visibility and temporal smoothing remain the biggest likely next gains. |
| `4_nearest_player_and_smoothing.ipynb` | Superseded champion. | Local player density and temporal smoothing improved both public and private MCC. | Ground contact remains weaker than player-player contact. |
| `5_type_specific_thresholds.ipynb` | Current scored champion. | Separate thresholds improved player-player precision and lifted both public and private MCC. | Ground contact did not improve, so it is the next target. |
| `6_type_specific_models.ipynb` | Rejected challenger. | Separate models slightly raised public MCC. | Private MCC dropped sharply and ground local MCC regressed. |
| `7_blended_type_models.ipynb` | Current challenger. | Blends unified and type-specific probabilities to avoid full ground-model replacement. | Submit only if local MCC and ground-slice gates both clear Notebook 5. |
| `8_yolo_video_feature_probe.ipynb` | Video/YOLO investigation. | Tests frame mapping, helmet overlay, optional YOLO detections, and cheap helmet-derived features. | It is not a submission notebook; use it to decide whether video features deserve a production model. |

## 15. Path Decision

Earlier notebooks carried several `DATA_DIR` candidates while the Kaggle mount
was still unclear. The correct path is now confirmed as:

```text
/kaggle/input/competitions/nfl-player-contact-detection
```

All notebooks should use that single path. This keeps Kaggle failures easier to
debug and avoids accidentally reading from an older dataset-style mount.
