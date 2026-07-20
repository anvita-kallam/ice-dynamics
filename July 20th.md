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
---
# Sequential VI Pipeline

To compare optimization strategies, a separate **sequential PINN → VI** training pipeline has been created alongside the existing joint training pipeline. This allows both approaches to run independently on the cluster without sharing checkpoints, logs, or outputs.

### New Files

| File | Purpose |
|------|---------|
| `Archive/train_vi_only_torch.py` | Stage 2 VI-only training using a frozen PINN and VGP-only Adam optimization |
| `Archive/predict_vi_only_torch.py` | Posterior prediction from VI-only checkpoints |
| `Archive/run_torch_sequential.cfg` | Configuration for Stage 1 PINN pretraining |
| `Archive/run_torch_vi_only.cfg` | Configuration for Stage 2 VI-only training |
| `Archive/slurm/vi_pretrain_sequential_more_sliding.sbatch` | Slurm submission script for sequential PINN pretraining |
| `Archive/slurm/vi_train_vi_only_more_sliding.sbatch` | Slurm submission script for VI-only training |
| `Archive/docs/SEQUENTIAL_VI_PIPELINE.md` | Documentation describing the sequential workflow and comparison with the joint pipeline |

### Reused Components

To minimize duplicated code, the sequential pipeline reuses the existing implementation wherever possible.

- `pretrain_solution_torch.py` for Stage 1 PINN pretraining
- `JointModel`, `joint_loss`, datasets, DDP utilities, and Slurm wrappers
- Helper functions from `train_torch.py`, including:
  - evaluation
  - η metrics
  - gradient clipping
  - pretrained model loading and validation
- Existing training metrics and plotting utilities, extended to support the VI-only workflow

### Isolated Output Paths

The sequential pipeline uses independent directories so it can run simultaneously with the joint pipeline.

| Resource | Joint Training | Sequential Training |
|----------|----------------|---------------------|
| Pretrain | `checkpoints/torch_pretrain/more_sliding/` | `checkpoints/torch_pretrain/more_sliding_sequential/` |
| Training | `checkpoints/torch_joint/more_sliding/` | `checkpoints/torch_vi_only/more_sliding/` |
| Log | `logs/log_train_torch_more_sliding` | `logs/log_train_vi_only_more_sliding` |
| Posterior Samples | `outputs/more_sliding_posterior_samples_torch.h5` | `outputs/more_sliding_vi_only_posterior_samples_torch.h5` |

### Running on the Cluster

```bash
cd ~/ice-dynamics/Archive

# Stage 1: Sequential PINN pretraining
sbatch slurm/vi_pretrain_sequential_more_sliding.sbatch

# Stage 2: VI-only training (after pretraining completes)
sbatch slurm/vi_train_vi_only_more_sliding.sbatch

# Existing joint training (unchanged)
sbatch slurm/vi_train_more_sliding.sbatch
```

Both the sequential and joint experiments can be submitted simultaneously on different GPUs.

### Frozen PINN Assumptions

- The VI-only stage (`restore=False`) loads the pretrained PINN from the sequential pretraining checkpoint (`model_best.pt`) and optionally verifies the observation loss.
- During Stage 2, the PINN remains completely frozen:
  - `requires_grad=False`
  - `eval()`
  - no PINN optimizer
  - no alternating optimization
- The VGP initializes from its prior unless resuming from an existing VI-only checkpoint (`checkpoints/torch_vi_only/more_sliding/model.pt`).
- `state_reg_scale=0` in `run_torch_vi_only.cfg`, since joint-training state regularization is unnecessary when the PINN is fixed.
```
## VI-Only Pipeline Updates

The sequential **VI-only** pipeline has been expanded to improve spatial viscosity recovery while keeping the joint training implementation completely independent.

### 1. VGP-Only Optimization

The VI-only training loop now optimizes only the VGP.

- PINN permanently set to `eval()`
- `requires_grad=False` for all PINN parameters
- Any accidental PINN gradients are cleared every iteration
- Removed joint and alternating optimization logic
- Only the VGP (and associated shift parameters) are updated

### 2. Configurable Gaussian Process

The sparse variational GP now supports configurable kernels while preserving the previous behavior by default.

New options include:

- `kernel_type`
  - `rbf`
  - `matern12`
  - `matern32`
  - `matern52`
- Anisotropic length scales
- Optional independent y-direction length scale (`l_scale_eta_y`)
- Learnable length scales
- Configurable inducing point count using ice-only farthest-point sampling

### 3. Optimizers and Learning Rate Schedules

The VI-only optimizer is now configurable.

Supported optimizers:

- Adam
- AdamW
- AdamW + Natural Gradient Descent (NGD)

Available learning rate schedulers:

- Cosine decay
- ReduceLROnPlateau
- Exponential decay
- None

Learning rate changes are logged throughout training.

### 4. Expanded Diagnostics

Additional metrics are now recorded during training.

Logged quantities include:

- `log10_eta_r`
- RMSE
- Bias
- Posterior standard deviation
- Calibration statistics
- Kernel length scales
- Number of inducing points
- ELBO components
- Gradient norms
- Learning rate

Additional plots visualize:

- Kernel parameters
- Posterior uncertainty

### 5. Early Stopping

Configurable early stopping has been added.

Example configuration:

- `early_stop_metric = log10_eta_r`
- `early_stop_patience = 80`

Training can now terminate automatically when the selected metric stops improving.

### 6. Hyperparameter Sweeps

Automated sweep utilities have been added for the VI-only pipeline.

Example commands:

```bash
python scripts/launch_vi_only_sweep.py configs/vi_only_sweep_grid.json

sbatch slurm/vi_train_vi_only_sweep_one.sbatch \
    configs/vi_only_sweeps/run_*.cfg
```

This enables systematic exploration of:

- η prior strength
- KL weight
- Learning rate
- Kernel type
- Inducing point count
- Other VI hyperparameters

### 7. Posterior Evaluation

A dedicated evaluation script has been added.

```bash
python evaluate_vi_only_posterior.py \
    run_torch_vi_only.cfg \
    --tag more_sliding
```

Outputs include:

- Posterior mean maps
- Uncertainty maps
- Error vs. uncertainty analysis
- 1σ / 2σ calibration
- Summary JSON statistics

### 8. Pipeline Isolation

The VI-only workflow remains fully independent of the joint training pipeline.

Separate resources are maintained for:

- Configuration files
- Checkpoints
- Logs
- Prediction outputs
- Evaluation results

All GP enhancements are implemented as optional arguments, so the existing joint pipeline is unaffected.

### Default VI-Only Configuration

The recommended starting configuration is:

- **Kernel:** Matérn 3/2
- **Inducing points:** 28 × 28 (ice-only FPS)
- **Optimizer:** AdamW
- **Learning rate scheduler:** Cosine decay
- **`eta_prior_scale`:** 0.08
- **Early stopping metric:** `log10_eta_r`
```
