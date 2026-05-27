# Coding Standards

## 1. Repository Scope

This repository is intentionally notebook-first. Kaggle notebooks are the
executable source of truth, while `docs/` captures competition instructions,
EDA findings, modeling decisions, and lightweight project notes.

Keep the root small:

- `notebooks/` for Kaggle notebooks.
- `docs/` for standards, instructions, EDA notes, and notebook writeups.
- `docs/3_model_evaluation_progress.md` for validation results,
  leaderboard scores, champion/challenger decisions, and next experiments.
- `README.md` for the high-level project overview and current best result.

Do not commit raw Kaggle data, local model checkpoints, generated videos,
feature caches, or Kaggle working directories.

## 2. Notebook Naming

Use numbered, stable notebook names that match the project workflow:

1. `1_eda_contact_tracking_video_context.ipynb`
2. `2_distance_baseline_first_experiment.ipynb`
3. `3_tracking_feature_model.ipynb`
4. `4_nearest_player_and_smoothing.ipynb`
5. `5_type_specific_thresholds.ipynb`
6. `6_type_specific_models.ipynb`

Future notebooks should continue the sequence and describe the actual workflow,
for example `7_video_features_and_fusion.ipynb` or
`8_model_ensemble.ipynb`.

## 3. Code Style

Follow PEP 8 for Python code:

- Use 4 spaces for indentation.
- Keep lines to 79 characters or fewer where practical.
- Prefer f-strings and small utility functions when they improve readability.
- Add type hints for reusable functions when the type is clear.
- Group imports as standard library, third-party libraries, then local imports.

Use Google-style docstrings for reusable functions that carry project logic.

## 4. Notebook Style

Each notebook should include:

- a short purpose statement at the top;
- a clear configuration section near the top;
- a `NOTEBOOK_VERSION` string printed by the first code cell, so Kaggle output
  can confirm the notebook is current;
- explicit mode flags such as `RUN_FAST`, `FAST_SAMPLE_PLAYS`, and
  `DISTANCE_THRESHOLDS`;
- the fixed Kaggle competition path
  `/kaggle/input/competitions/nfl-player-contact-detection`;
- Markdown insight cells after important checks, plots, or metrics;
- artifact-writing cells for reusable outputs such as `submission.csv`.

Prefer readable, self-contained notebook code over imports from local project
modules. Kaggle should be able to run each notebook with only the competition
dataset available at the fixed input path.

Notebook outputs are useful when they document a real Kaggle run:

- If notebook code has not changed, keep existing outputs so metrics,
  diagnostics, and tables remain reviewable in GitHub and the IDE.
- If notebook code changes, clear stale outputs before committing unless the
  notebook was rerun after the code change.
- New notebooks should be committed without outputs until they have a trusted
  Kaggle run.
- When reviewing notebook diffs, separate source changes from output-only
  refreshes.

Kaggle remains the trusted execution environment for final metrics.

Before running a notebook on Kaggle:

1. Pull or sync the latest repository/notebook version.
2. Run the first setup cell and check `NOTEBOOK_VERSION`.
3. Confirm the printed version matches the expected current version in
   `docs/3_model_evaluation_progress.md`.
4. Only trust metrics from the current version.

## 5. Validation Standards

The competition metric is Matthews Correlation Coefficient. Baseline and model
notebooks should report MCC on validation predictions and should be explicit
about the validation split.

Default validation strategy:

- split by `game_play` when validating row-level contact predictions;
- keep all rows from a play in the same fold;
- tune hard decision thresholds only on validation or out-of-fold predictions;
- report player-player and player-ground behavior separately when possible.

## 6. Feature Engineering Standards

Feature engineering must use only fields available at inference time unless a
target is explicitly being attached for validation.

Project-safe feature themes:

- player-player distance and relative motion from tracking coordinates;
- player-ground heuristics using speed, acceleration, orientation, and video;
- pair identity and roster-safe interaction context;
- helmet box geometry, visibility, and synchronized video-frame context;
- temporal smoothing within a play without using future labels.

Avoid target leakage:

- Do not use contact labels in feature construction.
- Do not use future labels or validation rows to tune row-level transforms.
- Keep game-play grouped validation for sequence-derived features.

## 7. Plot Style

Use the Viridis palette as the default visual language across notebooks:

- Use `sns.color_palette("viridis", ...)` for categorical or sequential
  accents.
- Use `"viridis"` as the default colormap for heatmaps.
- Keep chart titles short and analytical.

## 8. Git Hygiene

Do not commit:

- raw Kaggle competition data;
- local model checkpoints;
- generated submission files;
- generated annotated videos;
- large cached feature tables;
- Python caches or notebook checkpoints.
