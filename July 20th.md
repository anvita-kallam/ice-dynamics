# Current Training Update + Config Changes

## Objective

The long-freeze experiment showed that the VGP initially recovered meaningful spatial viscosity structure (`log10_eta_r ≈ 0.58`), but the spatial correlation gradually collapsed to nearly zero during the extended frozen phase. While the mean viscosity remained accurate, the combination of the η prior, KL regularization, and prolonged PINN freeze drove the VGP back toward an almost constant viscosity field.

## Updated Training Recipe

The training configuration has been adjusted to encourage the VGP to recover spatial structure while maintaining the stability achieved in previous runs.

| Knob | Previous | Updated | Motivation |
|------|---------:|--------:|------------|
| `eta_prior_scale` | 0.20 | **0.05** | Reduce the pull toward a constant viscosity field |
| `eta_prior_std` | 1.0 | **1.5** | Allow greater spatial variation in η |
| `kl_eta` | 0.50 | **0.15** | Weaken regularization on the GP posterior |
| `phys_scale` / `ssa_*_std` | 3 / 0.08 | **4 / 0.06** | Increase the influence of spatial physics |
| `vgp_eta_lr` | 5e-4 | **1e-3** | Encourage faster exploration of spatial modes |
| `mean_net_lr` | 1e-5 | **5e-6** | Keep the PINN relatively fixed |
| `vgp_steps_per_mean_step` | 2 | **3** | Give the VGP additional optimization steps |
| Optimizer | Adam | **Adam** | Retain the faster optimizer for experimentation |

### Current Goal

The objective is to determine whether the collapsed spatial correlation can recover without restarting the experiment.

Success criteria over the next **50–100 epochs**:

- `log10_eta_r` begins increasing again (target ≥ 0.2–0.3).
- `log10_bias` remains close to zero.
- The VGP regains meaningful spatial variation while preserving the correct mean viscosity.

If `log10_eta_r` remains near zero after approximately 100 additional epochs, it will likely indicate that recovering from a flattened VGP is difficult once the posterior has collapsed.

---

## Planned Fresh Training Run

In parallel, a new joint training recipe has been prepared based on observations from the previous experiments.

The primary change is to use a **much shorter PINN freeze**, since the highest spatial correlation consistently occurred early in training before gradually disappearing.

| Setting | Value | Motivation |
|---------|------:|------------|
| Freeze length | **180 epochs** | Capture the early spatial learning before flattening occurs |
| Total epochs | **700** | Fits within approximately 2–3 walltime allocations |
| PINN optimizer after unfreeze | **Adam** | Avoid the significant slowdown observed with L-BFGS |
| Alternating updates | **1 VGP : 1 PINN** | Faster training; can increase if needed |
| Inducing points | **24×24** | Balance spatial expressiveness and KL cost |
| `eta_prior_scale` / `kl_eta` / `phys_scale` | **0.05 / 0.15 / 4** | Encourage spatial viscosity recovery while maintaining stability |

Separate optimizers, independent learning rates, gradient clipping, and alternating optimization remain enabled. The only planned change is to continue using Adam throughout training rather than switching to L-BFGS.

### Expected Behavior

The new training run will be monitored for:

- **Epoch 100–180:** `log10_eta_r` increasing while the PINN is still frozen (target ≳ 0.3–0.5).
- **After epoch 180:** PINN unfreezes and begins joint optimization.
- If interrupted by walltime, training will resume from the checkpoint using the same configuration.

The expectation is that the shorter freeze will preserve the early spatial structure while allowing joint optimization to refine the solution without collapsing the viscosity field.
