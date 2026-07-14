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

### Configuration Changes

| Parameter | Old | New |
|-----------|-----|-----|
| `phys_scale` | 1 | **5** |
| `ssa_*_std` | 1.0 | **0.05** |
| `mean_net_lr` / `vgp_eta_lr` | 2e-5 / 5e-4 | **1e-5 / 2e-3** |
| Inducing points | 20×20 bounding box | **28×28 ice-masked farthest-point sampling (784 points)** |
| `restore` | True | **False** (fresh start due to changed VGP structure) |

- Implemented **ice-only farthest-point sampling (`ice_fps`)** for VGP inducing point placement in `models_torch.py`.
- Added debug logging for **η gradient updates** (`eta_log_shift`) and the **VGP-to-PINN gradient ratio**.
- Added a warning when **unfrozen PINN gradients exceed VGP gradients by more than 100×**, indicating the PINN is dominating optimization instead of η.

### Results

The updated training configuration appears to be working as intended.

| Metric | Previous | Current (Epoch 0) |
|--------|----------|-------------------|
| Configuration | `ssa_*_std = 1`, `phys_scale = 1` | `ssa_*_std = 0.05`, `phys_scale = 5`, 784 ice-masked inducing points |
| VGP η gradient | ~1e−6 | **~0.99** |
| `log10_bias` | ~−1.29 | **−0.22** |
| Mean predicted η | ~0.63 | **~7.5** (reference ≈ 15) |
| `log10_r` | ~0 | **~0.40** |


The stronger physics weighting, tighter SSA uncertainty, and improved inducing point placement significantly increased the gradient signal to the VGP. As a result, the optimizer is now updating η effectively, reducing the viscosity bias and improving correlation with the reference field from the start of training.

The large negative physics loss (≈ −20) is expected with `ssa_*_std = 0.05`, since the Gaussian negative log-likelihood becomes negative when residuals are small. The key indicator of success is that the η gradients are now substantial and the predicted viscosity is much closer to the reference values.
