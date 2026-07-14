# Continue Model Hyperparameter Tuning

## Objective

Improve joint PINN + Variational Inference training so that the model recovers the latent viscosity field (η) instead of minimizing the loss by modifying the PINN state variables (velocity, thickness, and surface elevation).

### Problem

Even with the PINN frozen during the first 400 epochs, the viscosity bias (`log10_bias`) remained around **−1.3**, indicating that the VGP was not effectively learning η. This suggested that the optimizer was not receiving a strong enough training signal from the physics loss, and viscosity updates remained too small to reduce the PDE residual.

### Changes Implemented

- Fixed the freeze schedule so it works correctly when resuming training.
- Added state regularization to keep the PINN close to its pretrained solution after unfreezing.
- Introduced separate learning rates for the PINN and η parameters.
- Added configurable weighting for the data, physics, and state regularization losses.
- Strengthened the physics constraint by increasing its influence during optimization.
- Expanded logging to track individual loss terms, gradient norms, and viscosity metrics (RMSE, bias, correlation) throughout training.
- Configured training to prioritize learning η first before jointly fine-tuning the PINN.
