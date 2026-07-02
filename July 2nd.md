# VI Modelling Test

## Objective: Develop a Variational Inference Training Pipeline

- Build an end-to-end workflow for inferring glacier viscosity from noisy surface velocity observations using Variational Inference (VI).

## VI Training Notebook Development

- Created `notebooks/learning/train_vi_viscosity_test.ipynb`.
- Implemented a workflow to:
  - Load test spin-up outputs from `TEST_CASES`.
  - Generate VI input bundles (`speed_obs`, masks, and ground-truth fields).
  - Train a field-based VI model to infer log-viscosity from noisy velocity observations.
  - Save posterior results to `outputs/vi/test/results_<case_id>.npz`.
  - Visualize posterior mean vs. ground-truth viscosity in both log and linear space.
  - Summarize inference results for both test cases.

## Variational Inference Model Implementation

- Created `scripts/vi_viscosity_model.py`.
- Implemented a prototype VI pipeline that:
  - Fits a local surrogate relating log-speed to log-viscosity using the forward-model snapshot.
  - Performs factorized Gaussian Variational Inference on the grounded ice domain using only noisy velocity observations.
  - Interpolates the inferred posterior back to the full glacier domain.
  - Returns posterior mean, posterior standard deviation, and evaluation metrics (R² and RMSE).

### Initial Model Performance

| Case | R² (log η) | RMSE (log η) |
|------|-----------:|-------------:|
| More Sliding | ~0.26 | ~0.64 |
| No Sliding | ~0.41 | ~0.82 |

## Key Takeaways

- Successfully established a complete prototype VI workflow from preprocessing through posterior inference and visualization.
- The current implementation is an emulator-based proof of concept using NumPy and SciPy, where the surrogate model replaces the forward physics during inference.
- A future physics-informed implementation will replace the surrogate likelihood with Icepack's `diagnostic_solve` to perform inference directly using the glacier dynamics model.

