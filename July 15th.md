# Continue Model Hyperparamater Tuning + Joint Training
## Objective

Stabilize joint PINN + Variational Inference training by preventing the viscosity field (η) from collapsing while maintaining meaningful physics optimization.

### Problem

The training job reached the walltime limit around **epoch 174**, but analysis showed that η had collapsed:

| Metric | Epoch 0 | Epoch ~174 |
|--------|---------|------------|
| Mean predicted η | ~7.5 | ~0.13 |
| `log10_bias` | −0.22 | −1.98 |
| Physics loss | ~−20.3 | ~−20.7 |

Although the physics loss improved slightly, the optimizer achieved this by driving η toward unrealistically small values while the PINN remained frozen. This indicated that the optimization was still minimizing the objective in the wrong direction, so resuming from the checkpoint was not appropriate.

### Changes Implemented

- Added a **soft η prior** centered at the initial viscosity (`eta_init`) to prevent the GP from drifting too far from physically reasonable values.
- Added a **hard lower bound** (`eta_min = 1.0`) to prevent viscosity collapse.
- Reduced the η learning rate (`2e-3 → 2e-4`) for more stable optimization.
- Increased KL regularization on η (`kl_eta = 0.5`) to keep the GP closer to its prior.
- Relaxed the physics loss (`phys_scale = 2`, `ssa_*_std = 0.1`) to reduce the incentive for unphysical η updates.
- Reduced the inducing point resolution from **28×28** to **20×20** ice-masked farthest-point sampling for faster training.
- Reset training with `restore = False` to start from a clean initialization rather than continuing the collapsed checkpoint.
- Added a warning when `log10_bias < -1` to detect viscosity collapse early during training.
