# Competition Instructions and Project Approach

## 1. Description

Kaggle's **1st and Future - Player Contact Detection** competition asks
competitors to detect external contact experienced by players during NFL plays.
The solution can use game video, player tracking data, baseline helmet
detections, and metadata to identify when contact occurs.

![Contact example](https://www.googleapis.com/download/storage/v1/b/kaggle-user-content/o/inbox%2F644036%2F65cd663d2c823043b36ecda6c93c1304%2Fcontact-example.gif?generation=1670265252697886&alt=media)

## 2. Goal

The goal is to predict all moments of contact at 10 Hz during a football play.
Contact can occur in two ways:

- between two players;
- between a player's body and the ground.

The target is binary:

- `contact = 1`: external player-player or player-ground contact occurred.
- `contact = 0`: no contact occurred for that row.

Accurate contact detection supports player safety analysis. The NFL and AWS
want to improve injury surveillance and mitigation by combining tracking and
video signals, including ground contact, which is not fully captured by
tracking-only approaches.

## 3. Context

The NFL has previously challenged Kaggle competitors to detect and identify
helmet impacts. This competition expands the objective from helmet impact to
all contact moments. The competition is part of the NFL and AWS Digital Athlete
work, which aims to build a detailed representation of player experience so
health and safety teams can better understand, prevent, and respond to injury
risk.

More complete contact labels can help connect specific types of contact with
injury outcomes and unsafe situations. For this project, the practical modeling
implication is that a single distance threshold is not enough: player-player
contact, player-ground contact, video visibility, timing alignment, and label
noise each need dedicated analysis.

## 4. Dataset Overview

Each play has associated videos and tabular data:

- Sideline and Endzone videos are time synced and aligned.
- All29 video is provided but is not guaranteed to be time synced.
- Videos have a frame rate of `59.94 Hz`.
- The snap occurs 5 seconds into the video, around frame `300`.
- Tracking and labels are sampled at `10 Hz`.

The training videos live in `train/`, and the scored videos live in `test/`.
The public test videos are mock plays copied from training and are not used for
final scoring. During submission, the notebook is rerun on a holdout test set
of unseen plays.

![Tracking orientation reference](https://www.googleapis.com/download/storage/v1/b/kaggle-user-content/o/inbox%2F3258%2F820e86013d48faacf33b7a32a15e814c%2FIncreasing%20Dir%20and%20O.png?generation=1572285857588233&alt=media)

## 5. Expected Kaggle Path

The notebooks use the known competition mount directly:

```text
/kaggle/input/competitions/nfl-player-contact-detection
```

## 6. Files

| File | Purpose |
| --- | --- |
| `train/`, `test/` | MP4 videos for each play. Main views are `Sideline` and `Endzone`; `All29` is additional and not guaranteed time synced. |
| `train_labels.csv` | Training labels for every player-player and player-ground candidate at each 0.1 second step. |
| `sample_submission.csv` | Required submission schema and rows. |
| `train_baseline_helmets.csv`, `test_baseline_helmets.csv` | Imperfect baseline helmet boxes and player assignments for Sideline and Endzone views. |
| `train_player_tracking.csv`, `test_player_tracking.csv` | 10 Hz player tracking data from sensors. |
| `train_video_metadata.csv`, `test_video_metadata.csv` | Video timing metadata used to sync video frames with tracking timestamps. |

## 7. `train_labels.csv`

| Column | Meaning |
| --- | --- |
| `contact_id` | Combination of `game_play`, `step`, `nfl_player_id_1`, and `nfl_player_id_2`. |
| `game_play` | Unique game and play identifier. |
| `nfl_player_id_1` | Lower numbered player ID in a player-player pair; the player ID for ground contact. |
| `nfl_player_id_2` | Higher numbered player ID in a player-player pair; `G` for ground contact. |
| `step` | Timestep within the play, starting at 0 and incrementing every 0.1 seconds. |
| `datetime` | 10 Hz timestamp of the contact label. |
| `contact` | Binary target. |

Labels may be imperfect and are expected to be approximately within one 10 Hz
timestep of the true contact moment. Validation and smoothing should account
for this noise.

## 8. `sample_submission.csv`

| Column | Meaning |
| --- | --- |
| `contact_id` | Required test row ID, following the same structure as labels. |
| `contact` | Binary prediction, where `1` means contact and `0` means no contact. |

The final file must be named `submission.csv`.

## 9. Baseline Helmet Files

| Column | Meaning |
| --- | --- |
| `game_play` | Unique game and play identifier. |
| `game_key` | Game ID. |
| `play_id` | Play ID. |
| `view` | `Sideline` or `Endzone`. |
| `video` | Associated video filename. |
| `frame` | Video frame number. |
| `nfl_player_id` | Imperfect predicted player ID. |
| `player_label` | Team side and jersey label. |
| `left`, `width`, `top`, `height` | Helmet bounding box. |

The helmet boxes come from a prior winning player-assignment model. They are
useful but should be treated as noisy inputs rather than ground truth.

## 10. Player Tracking Files

| Column | Meaning |
| --- | --- |
| `game_play`, `game_key`, `play_id` | Play identifiers. |
| `nfl_player_id` | Player ID. |
| `datetime` | 10 Hz timestamp. |
| `step` | Timestep relative to play start. |
| `position` | Football position. |
| `team` | Home or away. |
| `jersey_number` | Jersey number. |
| `x_position`, `y_position` | Player location on the field in yards. |
| `speed` | Speed in yards per second. |
| `distance` | Distance traveled from the prior timestep in yards. |
| `orientation` | Player orientation in degrees. |
| `direction` | Motion direction in degrees. |
| `acceleration` | Total acceleration in yards per second squared. |
| `sa` | Signed acceleration in the direction of motion. |

## 11. Video Metadata Files

| Column | Meaning |
| --- | --- |
| `game_play`, `game_key`, `play_id` | Play identifiers. |
| `view` | `Sideline` or `Endzone`. |
| `start_time` | Video start timestamp. |
| `end_time` | Video end timestamp. |
| `snap_time` | Timestamp when the play starts within the video. |

Note that metadata files contain timing fields, while the baseline helmet files
contain video filenames. The EDA notebook handles this distinction explicitly.

## 12. Evaluation

Submissions are evaluated with Matthews Correlation Coefficient between
predicted and actual contact labels. MCC is appropriate here because the target
is highly imbalanced and accuracy would be misleading.

Submission format:

```text
contact_id,contact
58168_003392_0_38590_43854,0
58168_003392_0_38590_41257,1
```

## 13. Code Competition Requirements

Final submissions must be Kaggle notebooks:

- CPU notebook runtime must be `<= 9 hours`.
- GPU notebook runtime must be `<= 9 hours`.
- Internet must be disabled.
- Publicly available external data and pretrained models are allowed when
  attached through Kaggle-compatible inputs.
- The submission file must be named `submission.csv`.

## 14. Project Plan

The repository starts with two notebooks:

1. Run EDA in
   [`1_eda_contact_tracking_video_context.ipynb`](../notebooks/1_eda_contact_tracking_video_context.ipynb)
   to understand labels, tracking, helmets, metadata, and baseline failure
   modes.
2. Build the first submission-safe experiment in
   [`2_distance_baseline_first_experiment.ipynb`](../notebooks/2_distance_baseline_first_experiment.ipynb)
   using the starter notebook's tracking-distance idea with cleaner validation.
3. Run the improved tracking-feature model in
   [`3_tracking_feature_model.ipynb`](../notebooks/3_tracking_feature_model.ipynb)
   to learn player-player and player-ground contact patterns from tracking
   features and tune the final threshold for MCC.
4. Run the nearest-player and smoothing challenger in
   [`4_nearest_player_and_smoothing.ipynb`](../notebooks/4_nearest_player_and_smoothing.ipynb)
   to test local-density features and temporal smoothing against the current
   tracking-feature champion.
5. Run the type-specific threshold challenger in
   [`5_type_specific_thresholds.ipynb`](../notebooks/5_type_specific_thresholds.ipynb)
   to tune separate ground and player-player cutoffs after smoothing.
6. Run the type-specific model challenger in
   [`6_type_specific_models.ipynb`](../notebooks/6_type_specific_models.ipynb)
   to train separate ground and player-player classifiers before smoothing and
   threshold tuning.
7. Run the blended type-model challenger in
   [`7_blended_type_models.ipynb`](../notebooks/7_blended_type_models.ipynb)
   to blend unified and type-specific probabilities while guarding the
   ground-contact slice.

Recommended next experiments:

1. Add relative distance, relative speed, acceleration, orientation, and nearest
   player features for player-player contact.
2. Build a dedicated ground-contact branch using motion, posture proxies,
   helmet boxes, and temporal context.
3. Add temporal smoothing by play and pair after out-of-fold validation.
4. Add video-derived features once the tracking-only validation harness is
   stable.
