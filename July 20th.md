# Joint Training Checkup + Sequential Train Setup

## 1. Joint Training: Lessons Learned

The latest joint training experiments confirmed that the optimization is now stable and successfully prevents viscosity collapse. However, a new limitation emerged.

During training, the VGP initially recovered meaningful spatial viscosity structure (`log10_eta_r ≈ 0.58`), but the correlation gradually collapsed toward zero during the prolonged frozen phase. Although the mean viscosity remained accurate, the combination of the η prior, KL regularization, and long PINN freeze encouraged the VGP to converge toward an almost constant viscosity field.

### Updated Joint Training Recipe

To encourage spatial recovery while preserving the current stability, the following hyperparameters were adjusted.

| Parameter | Previous | Updated | Purpose |
|------------|---------:|--------:|---------|
| `eta_prior_scale` | 0.20 | **0.05** | Reduce pull toward a constant viscosity field |
| `eta_prior_std` | 1.0 | **1.5** | Allow greater spatial variation |
| `kl_eta` | 0.50 | **0.15** | Weaken GP regularization |
| `phys_scale` / `ssa_*_std` | 3 / 0.08 | **4 / 0.06** | Strengthen spatial physics signal |
| `vgp_eta_lr` | 5e-4 | **1e-3** | Encourage faster exploration of spatial modes |
| `mean_net_lr` | 1e-5 | **5e-6** | Keep the PINN nearly fixed |
| `vgp_steps_per_mean_step` | 2 | **3** | Increase VGP influence during training |
| Optimizer | Adam | **Adam** | Retain the faster optimizer |

### Current Objective

The goal is to determine whether the collapsed spatial correlation can recover without restarting the experiment.

Success criteria over the next 50–100 epochs include:

- `log10_eta_r` begins increasing again.
- `log10_bias` remains close to zero.
- The VGP regains meaningful spatial variation while preserving the correct mean viscosity.

If `log10_eta_r` remains near zero after roughly 100 additional epochs, it will likely indicate that recovering from a flattened posterior is difficult once the VGP has converged to a nearly constant solution.

### Planned Fresh Joint Run

A second joint-training configuration has also been prepared based on observations from previous experiments.

| Setting | Value | Motivation |
|---------|------:|------------|
| Freeze length | **180 epochs** | Capture early spatial learning before flattening occurs |
| Total epochs | **700** | Fits within approximately 2–3 walltime allocations |
| PINN optimizer after unfreeze | **Adam** | Avoid slowdown observed with L-BFGS |
| Alternating updates | **1 VGP : 1 PINN** | Faster optimization; can increase if needed |
| Inducing points | **24×24** | Balance spatial expressiveness and computational cost |
| `eta_prior_scale` / `kl_eta` / `phys_scale` | **0.05 / 0.15 / 4** | Encourage spatial recovery while maintaining stability |

Separate optimizers, independent learning rates, gradient clipping, and alternating optimization remain enabled. The only planned change is to continue using Adam throughout training rather than switching to L-BFGS.

The new run will be monitored to verify that spatial correlation develops during the shortened freeze and remains stable after joint optimization begins.

---

# 2. Sequential PINN → VI Pipeline

To determine whether the remaining limitation is caused by joint optimization or by the inverse problem itself, a completely independent **sequential PINN → VI** pipeline has been developed.

The workflow consists of:

1. Train the PINN to convergence.
2. Freeze the pretrained PINN.
3. Perform Variational Inference using only the VGP to infer viscosity.

The sequential implementation is fully isolated from the joint pipeline, allowing both experiments to run simultaneously without sharing checkpoints, logs, or outputs.

## New Files

| File | Purpose |
|------|---------|
| `Archive/train_vi_only_torch.py` | Stage 2 VI-only training using a frozen PINN and VGP-only optimization |
| `Archive/predict_vi_only_torch.py` | Posterior prediction from VI-only checkpoints |
| `Archive/run_torch_sequential.cfg` | Stage 1 PINN pretraining configuration |
| `Archive/run_torch_vi_only.cfg` | Stage 2 VI-only configuration |
| `Archive/slurm/vi_pretrain_sequential_more_sliding.sbatch` | Slurm script for sequential pretraining |
| `Archive/slurm/vi_train_vi_only_more_sliding.sbatch` | Slurm script for VI-only training |
| `Archive/docs/SEQUENTIAL_VI_PIPELINE.md` | Documentation comparing the sequential and joint workflows |

## Reused Components

To minimize duplicated code, the sequential implementation reuses the existing infrastructure wherever possible.

- `pretrain_solution_torch.py`
- `JointModel`
- `joint_loss`
- datasets
- DDP utilities
- Slurm wrappers
- evaluation utilities
- η metrics
- gradient clipping
- pretrained checkpoint loading
- plotting and training metrics

## Independent Output Paths

| Resource | Joint Training | Sequential Training |
|----------|----------------|---------------------|
| Pretraining | `checkpoints/torch_pretrain/more_sliding/` | `checkpoints/torch_pretrain/more_sliding_sequential/` |
| Training | `checkpoints/torch_joint/more_sliding/` | `checkpoints/torch_vi_only/more_sliding/` |
| Logs | `logs/log_train_torch_more_sliding` | `logs/log_train_vi_only_more_sliding` |
| Posterior Samples | `outputs/more_sliding_posterior_samples_torch.h5` | `outputs/more_sliding_vi_only_posterior_samples_torch.h5` |

## Running on the Cluster

```bash
cd ~/ice-dynamics/Archive

# Stage 1: Sequential PINN pretraining
sbatch slurm/vi_pretrain_sequential_more_sliding.sbatch

# Stage 2: VI-only optimization
sbatch slurm/vi_train_vi_only_more_sliding.sbatch

# Existing joint training
sbatch slurm/vi_train_more_sliding.sbatch
```

Both workflows can be submitted simultaneously on separate GPUs.

---

# 3. VI-Only Pipeline Improvements

After creating the sequential pipeline, several improvements were implemented specifically for the VI stage to maximize spatial viscosity recovery.

## VGP-Only Optimization

The VI-only training loop now optimizes only the VGP.

- PINN permanently in `eval()` mode
- `requires_grad=False` for all PINN parameters
- Any accidental PINN gradients cleared every iteration
- Joint and alternating optimization removed
- Only the VGP (and associated shift parameters) are updated

## Configurable Gaussian Process

The sparse variational GP now supports configurable kernels while preserving previous behavior by default.

Available options include:

- RBF kernel
- Matérn 1/2
- Matérn 3/2
- Matérn 5/2
- Anisotropic length scales
- Optional independent y-direction length scale
- Learnable length scales
- Configurable inducing point count using ice-only farthest-point sampling

## Optimizers and Learning Rate Scheduling

The VI optimizer is now fully configurable.

Supported optimizers:

- Adam
- AdamW
- AdamW + Natural Gradient Descent (NGD)

Supported schedulers:

- Cosine decay
- ReduceLROnPlateau
- Exponential decay
- None

Learning-rate changes are logged throughout training.

## Expanded Diagnostics

Training now records:

- `log10_eta_r`
- RMSE
- Bias
- Posterior standard deviation
- Calibration statistics
- Kernel length scales
- Number of inducing points
- ELBO decomposition
- Gradient norms
- Learning rate

Additional plots visualize:

- Kernel evolution
- Posterior uncertainty

## Early Stopping

Configurable early stopping has been added.

Example:

```text
early_stop_metric = log10_eta_r
early_stop_patience = 80
```

Training can terminate automatically once the selected metric stops improving.

## Hyperparameter Sweeps

A sweep utility has been added to automate large parameter studies.

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

## Posterior Evaluation

A dedicated evaluation script now analyzes posterior quality.

```bash
python evaluate_vi_only_posterior.py \
    run_torch_vi_only.cfg \
    --tag more_sliding
```

Outputs include:

- Posterior mean maps
- Posterior uncertainty maps
- Error vs. uncertainty analysis
- 1σ / 2σ calibration
- Summary statistics

## Default VI-Only Configuration

The recommended starting configuration is:

- Kernel: Matérn 3/2
- Inducing points: 28 × 28 (ice-only FPS)
- Optimizer: AdamW
- Learning-rate scheduler: Cosine decay
- `eta_prior_scale = 0.08`
- Early stopping on `log10_eta_r`
```
