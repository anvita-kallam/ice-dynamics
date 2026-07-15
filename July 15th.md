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

## Model Tuning and Debugging

After the initial training results showed that the model was minimizing the loss by modifying the PINN state (velocity and thickness) instead of learning the latent viscosity field (η), several iterations of debugging and model tuning were performed. The objective of these changes was to force the optimizer to recover physically meaningful viscosity while preserving the pretrained glacier state.


```text
                      Initial Training Results
        ┌─────────────────────────────────────────────┐
        │ • Poor η recovery                           │
        │ • log10_bias ≈ -1                           │
        │ • log10_r ≈ 0                              │
        │ • PINN minimized loss by changing          │
        │   velocity/thickness instead of viscosity  │
        └─────────────────────────────────────────────┘
                              │
                              ▼
                  Diagnose Root Cause
        ┌─────────────────────────────────────────────┐
        │ • PINN dominated optimization              │
        │ • Physics loss too weak                    │
        │ • η gradients ≈ 0                          │
        │ • Resume bug bypassed freeze               │
        │ • No constraint on pretrained PINN         │
        └─────────────────────────────────────────────┘
                              │
                              ▼
                 First Round of Improvements
        ┌─────────────────────────────────────────────┐
        │ ✓ Resume-safe freezing                     │
        │ ✓ State regularization                     │
        │ ✓ Separate learning rates                  │
        │ ✓ Stronger physics weighting               │
        │ ✓ Better logging & diagnostics             │
        │ ✓ Ice-only inducing point placement        │
        └─────────────────────────────────────────────┘
                              │
                              ▼
                     Initial Improvement
        ┌─────────────────────────────────────────────┐
        │ η gradients: 1e-6 → ~1                     │
        │ log10_bias: -1.29 → -0.22                  │
        │ log10_r: 0 → 0.40                          │
        │ η predictions much closer to reference     │
        └─────────────────────────────────────────────┘
                              │
                              ▼
                 New Problem Discovered
        ┌─────────────────────────────────────────────┐
        │ After ~174 epochs:                         │
        │ • η collapsed toward zero                  │
        │ • log10_bias → -1.98                       │
        │ • Physics loss improved by reducing η      │
        │   instead of learning correct viscosity    │
        └─────────────────────────────────────────────┘
                              │
                              ▼
                Second Round of Improvements
        ┌─────────────────────────────────────────────┐
        │ ✓ Added η prior anchor                     │
        │ ✓ Added hard η minimum                     │
        │ ✓ Reduced η learning rate                  │
        │ ✓ Increased KL regularization              │
        │ ✓ Relaxed physics weighting                │
        │ ✓ Restarted from fresh checkpoint          │
        │ ✓ Early collapse detection warnings        │
        └─────────────────────────────────────────────┘
                              │
                              ▼
                         Current Goal
        ┌─────────────────────────────────────────────┐
        │ Learn physically realistic viscosity (η)   │
        │ while preserving the pretrained PINN and   │
        │ satisfying the governing physics.          │
        └─────────────────────────────────────────────┘
```
---
