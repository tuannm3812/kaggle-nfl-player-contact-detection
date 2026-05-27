# Model Evaluation Progress

## 1. Current Champion

Current expected notebook versions:

| Notebook | Expected Version |
| --- | --- |
| `1_eda_contact_tracking_video_context.ipynb` | `EDA_V9_CANONICAL_PATH` |
| `2_distance_baseline_first_experiment.ipynb` | `DISTANCE_BASELINE_V9_CANONICAL_PATH` |
| `3_tracking_feature_model.ipynb` | `TRACKING_FEATURE_V5_DYNAMICS` |
| `4_nearest_player_and_smoothing.ipynb` | `NEAREST_SMOOTHING_V1_CHALLENGER` |
| `5_type_specific_thresholds.ipynb` | `TYPE_THRESHOLDS_V2_SAFE_MCC` |
| `6_type_specific_models.ipynb` | `TYPE_MODELS_V1_CHALLENGER` |
| `7_blended_type_models.ipynb` | `BLENDED_TYPE_MODELS_V1_CHALLENGER` |

If Kaggle output shows an older version string, sync the notebook before using
the output for decisions.

| Rank | Notebook | Submission Name | Local Validation MCC | Public MCC | Private MCC | Decision |
| ---: | --- | --- | ---: | ---: | ---: | --- |
| 1 | `5_type_specific_thresholds.ipynb` | Type-Specific Thresh, Version 3 | 0.67650 | 0.65170 | 0.65127 | Current scored champion |
| 2 | `7_blended_type_models.ipynb` | Blended type models | Pending | Pending | Pending | Run next |
| 3 | `6_type_specific_models.ipynb` | Type-Specific Model, Version 2 | 0.67530 | 0.65212 | 0.64025 | Rejected: private regression |
| 4 | `4_nearest_player_and_smoothing.ipynb` | Nearest Player, Version 3 | 0.67455 | 0.64497 | 0.64763 | Superseded |
| 5 | `3_tracking_feature_model.ipynb` | Tracking Feature, Version 3 | 0.65310 | 0.63075 | 0.62593 | Superseded |
| 6 | `2_distance_baseline_first_experiment.ipynb` | Distance baseline | 0.51863 | - | - | Superseded |

Notebook 3 is the first scored model. Its public/private gap is small
(`0.00482`), which suggests the model generalizes reasonably across leaderboard
splits. Local validation is optimistic by about `0.022` to `0.027` MCC.

Notebook 5 is now the scored champion. It improved over Notebook 4 by
`+0.00673` public MCC and `+0.00364` private MCC.

## 2. Champion Diagnostics

Notebook 3 local validation by contact type:

| Contact Type | Local MCC | Actual Rate | Predicted Rate |
| --- | ---: | ---: | ---: |
| Ground | 0.41231 | 3.47% | 2.90% |
| Player-player | 0.71656 | 1.02% | 1.36% |

Interpretation:

- Player-player contact is the model's strongest slice.
- Ground contact is the biggest weakness and should drive the next feature
  work.
- The model's validation is useful for ranking ideas, but public/private scores
  should be used to confirm final submission decisions.

## 3. Active Challenger

| Notebook | Status | Goal | Submit If |
| --- | --- | --- | --- |
| `7_blended_type_models.ipynb` | Ready to run | Blend the stable unified model with type-specific models before smoothing and threshold tuning | Submit only if local MCC beats `0.67650` and ground MCC is at least `0.50623` |

Notebook 4 targeted two likely weaknesses:

- local density: contact risk depends on nearest teammate/opponent context;
- label noise: contact labels may be offset by roughly one 10 Hz timestep, so
  probability smoothing can help if the threshold is retuned on held-out plays.

Notebook 4 validation details:

| Metric | Value |
| --- | ---: |
| Best smoothing window | 5 |
| Best threshold | 0.59 |
| Local validation MCC | 0.67455 |
| Precision | 0.59742 |
| Recall | 0.77176 |
| F1 | 0.67349 |
| Predicted positive rate | 1.59% |
| Submission positive rate | 2.09% |

Notebook 4 local validation by contact type:

| Contact Type | Local MCC | Actual Rate | Predicted Rate |
| --- | ---: | ---: | ---: |
| Ground | 0.50623 | 3.47% | 3.87% |
| Player-player | 0.72226 | 1.02% | 1.37% |

Notebook 5 improved this by keeping the ground threshold at `0.59` and raising
the player-player threshold to `0.70`. The gain came from player-player
precision: player-player local MCC rose from `0.72226` to `0.72836`, while
ground local MCC stayed at `0.50623`.

Notebook 5 validation details:

| Metric | Value |
| --- | ---: |
| Best smoothing window | 5 |
| Best ground threshold | 0.59 |
| Best player-player threshold | 0.70 |
| Local validation MCC | 0.67650 |
| Predicted positive rate | 1.38% |
| Submission positive rate | 1.92% |

Notebook 5 local validation by contact type:

| Contact Type | Local MCC | Actual Rate | Predicted Rate |
| --- | ---: | ---: | ---: |
| Ground | 0.50623 | 3.47% | 3.87% |
| Player-player | 0.72836 | 1.02% | 1.15% |

Notebook 6 tested full model separation. It raised public MCC slightly but
dropped private MCC sharply, so it should not be selected:

| Metric | Notebook 5 | Notebook 6 |
| --- | ---: | ---: |
| Local validation MCC | 0.67650 | 0.67530 |
| Public MCC | 0.65170 | 0.65212 |
| Private MCC | 0.65127 | 0.64025 |
| Ground local MCC | 0.50623 | 0.49675 |
| Player-player local MCC | 0.72836 | 0.72869 |
| Submission positive rate | 1.92% | 2.04% |

Interpretation: Notebook 6 mostly preserved player-player behavior, but the
ground model became less stable. The public/private split suggests overfitting
to the public holdout or a distribution mismatch in the separated ground model.

Notebook 7 tests a safer version of this idea by blending Notebook 5's unified
probabilities with Notebook 6-style type-specific probabilities. The goal is to
capture any player-player lift without allowing the ground model to fully
replace the more stable unified model.

## 4. Evaluation Rules

Use this decision order for new experiments:

1. Compare grouped local validation MCC against the current champion.
2. Check contact-type slices, especially ground-contact MCC.
3. Check predicted positive rate against the validation positive rate.
4. Submit only if validation improves or the slice tradeoff is clearly useful.
5. After submission, record public and private MCC here before starting the
   next experiment.

## 5. Backlog

| Priority | Experiment | Reason |
| ---: | --- | --- |
| 1 | Run Notebook 7 | Tests whether partial blending keeps Notebook 5 stability while using any useful type-model signal. |
| 2 | Dedicated ground-contact features | Ground slice remains weaker and Notebook 6 made it less stable. |
| 3 | Helmet visibility features | Baseline helmet boxes can provide posture and visibility proxies. |
| 4 | Short-window features around `t-2` to `t+2` | Contact labels are noisy and contact evolves over adjacent steps. |
| 5 | Model blending | A ground-focused model may complement the tracking champion. |
