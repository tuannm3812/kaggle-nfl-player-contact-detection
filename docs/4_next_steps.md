# Next Steps

This roadmap captures the practical next actions for improving the NFL Player
Contact Detection project. Model scores and champion decisions are tracked in
[`3_model_evaluation_progress.md`](3_model_evaluation_progress.md).

## 1. Immediate Actions

1. **Submit Notebook 7**

   Run `7_blended_type_models.ipynb` on Kaggle and submit it as the next
   leaderboard challenger. It has the strongest current local MCC:
   `0.67944`.

2. **Run Notebook 9**

   Run `9_helmet_feature_model.ipynb` to test whether helmet visibility, box
   geometry, and helmet-pair pixel distance improve local validation beyond
   Notebook 7.

3. **Record public/private scores**

   After every Kaggle submission, update
   [`3_model_evaluation_progress.md`](3_model_evaluation_progress.md) with local,
   public, and private MCC before starting the next experiment.

## 2. Modeling Priorities

| Priority | Experiment | Goal |
| ---: | --- | --- |
| 1 | Notebook 7 submission | Confirm whether the local blend gain generalizes. |
| 2 | Notebook 9 helmet features | Add player-assigned video context without relying on generic YOLO detections. |
| 3 | Short-window helmet interpolation | Fill missing helmet boxes and capture contact timing drift around `t-2` to `t+2`. |
| 4 | Multi-fold grouped validation | Reduce single-split volatility before heavier modeling. |
| 5 | Stage-2 blend | Blend tracking probabilities with selected distance, position, and video-derived features. |
| 6 | YOLOv7/segmentation research | Revisit only if helmet-feature modeling plateaus. |

## 3. Acceptance Gates

A new model should be promoted only when it clears these checks:

1. Local validation MCC improves over the current local challenger.
2. Ground-contact MCC does not regress materially.
3. Player-player precision remains stable.
4. Predicted positive rate stays close to validation and prior submissions.
5. Public/private leaderboard scores confirm the local trend.

## 4. Recommended Notebook Order

1. Run `7_blended_type_models.ipynb`.
2. Run `9_helmet_feature_model.ipynb`.
3. If Notebook 9 improves local validation, submit it.
4. If Notebook 9 does not improve, reduce helmet features to the strongest
   visibility and pixel-distance signals and retry.
5. Add short-window helmet interpolation only after the exact-frame helmet
   feature set is understood.

## 5. Research Notes

Generic YOLOv8 ran successfully in research mode, but on the inspected football
frame it detected only a few objects while the competition-provided helmet file
had dense player-assigned boxes. For this project, helmet boxes are currently
the better production feature source. YOLO remains useful for qualitative review
and possible future body-mask experiments.
