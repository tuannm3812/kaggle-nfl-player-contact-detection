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

If Kaggle output shows an older version string, sync the notebook before using
the output for decisions.

| Rank | Notebook | Submission Name | Local Validation MCC | Public MCC | Private MCC | Decision |
| ---: | --- | --- | ---: | ---: | ---: | --- |
| 1 | `4_nearest_player_and_smoothing.ipynb` | Nearest Player, Version 3 | 0.67455 | 0.64497 | 0.64763 | Current scored champion |
| 2 | `5_type_specific_thresholds.ipynb` | Type-specific thresholds | Pending | Pending | Pending | Run next |
| 3 | `3_tracking_feature_model.ipynb` | Tracking Feature, Version 3 | 0.65310 | 0.63075 | 0.62593 | Superseded |
| 4 | `2_distance_baseline_first_experiment.ipynb` | Distance baseline | 0.51863 | - | - | Superseded |

Notebook 3 is the first scored model. Its public/private gap is small
(`0.00482`), which suggests the model generalizes reasonably across leaderboard
splits. Local validation is optimistic by about `0.022` to `0.027` MCC.

Notebook 4 is now the scored champion. It improved over Notebook 3 by
`+0.01422` public MCC and `+0.02170` private MCC.

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
| `5_type_specific_thresholds.ipynb` | Fixed; rerun needed | Tune separate ground and player-player thresholds after smoothing | Submit only if local MCC beats `0.67455` |

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

Notebook 5 tests whether separate contact-type thresholds can improve on this
without changing the model itself.

Latest Notebook 5 run reached model training but failed during threshold
tuning. The model output is therefore not a submission candidate yet.

| Failed Version | Error | Fix |
| --- | --- | --- |
| `TYPE_THRESHOLDS_V1_CHALLENGER` | `np.sqrt` failed on large Python/object confusion-count products in the custom MCC helper. | `TYPE_THRESHOLDS_V2_SAFE_MCC` uses `math.sqrt` with float factors and clears the failed output. |

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
| 1 | Run Notebook 5 | Tests whether type-specific thresholds improve MCC over Notebook 4. |
| 2 | Dedicated ground-contact model | Ground slice remains weaker than player-player contact. |
| 3 | Helmet visibility features | Baseline helmet boxes can provide posture and visibility proxies. |
| 4 | Short-window features around `t-2` to `t+2` | Contact labels are noisy and contact evolves over adjacent steps. |
| 5 | Model blending | A ground-focused model may complement the tracking champion. |
