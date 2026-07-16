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
---

# Since July 9th: Model Tuning and Debugging

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
## 1. Updated Training Recipe (`run_torch.cfg`)

### Objective

Adjust the overall training configuration so that the optimizer receives a stronger—but stable—signal to learn viscosity.

### Problem

The previous configuration made the optimizer either:

- ignore viscosity completely (physics too weak), or
- drive viscosity toward zero (physics too strong).

Neither produced physically meaningful η estimates.

### Changes

```ini
[prior]
eta_init = 12.0
eta_min = 1.0
kl_eta = 0.5
kl_lambda = 0.0
num_inducing_x = 20
num_inducing_y = 20
inducing_placement = 'ice_fps'

[likelihood]
ssa_rx_std = 0.1
ssa_ry_std = 0.1
ssa_rh_std = 0.1

[train]
mean_net_lr = 1e-5
vgp_eta_lr = 2e-4
freeze_mean_net_epochs = 400
phys_scale = 2.0
state_reg_scale = 1.0
eta_prior_scale = 2.0
eta_prior_std = 1.0
```

### Explanation

Think of training as balancing several competing goals.

This configuration changes:

- **how strongly physics is enforced**
- **how quickly viscosity is allowed to change**
- **how long the PINN remains frozen**
- **how strongly viscosity is encouraged to remain physically realistic**

The new values were chosen because earlier experiments showed that overly aggressive optimization caused η to collapse toward zero.

---

## 2. Added a Soft η Prior

### Objective

Prevent viscosity from collapsing to unrealistically small values.

### Problem

The optimizer discovered that making η extremely small slightly improved the physics loss, even though the resulting viscosity field was physically meaningless.

### Code

```python
eta_loc = self.vgp_eta.mean(Xn)
log_eta_offset = self.eta_log_shift + eta_loc

eta_prior_reg = 0.5 * torch.mean(
    (log_eta_offset / eta_prior_std) ** 2
)
```

The prior penalty is combined with the KL divergence:

```python
kl_and_prior = kl_value + eta_prior_scale * eta_prior_reg
```

### Explanation

It gently encourages viscosity to remain near its initial physically reasonable value while still allowing it to move if the data supports it.

---

## 3. Added State Regularization

### Objective

Prevent the PINN from changing velocity and thickness to explain the observations.

### Problem

Even after unfreezing, the optimizer preferred modifying

- velocity
- thickness
- surface elevation

instead of recovering viscosity.

### Code

```python
with torch.no_grad():
    ur, vr, sr, hr = self.mean_net_ref(...)

state_u = batch_obs["uv_mask"] * (up - ur).square()
...
state_reg = ...
```

### Explanation

The pretrained PINN already produces a physically reasonable glacier.

State regularization tells the optimizer:

> "Stay close to this solution unless there is a good reason to move."

This forces the optimizer to explain remaining errors through η instead of rewriting the glacier state.

---

## 4. Separate ELBO Loss Components

### Objective

Understand which part of the loss dominates optimization.

### Problem

Previously only the total loss was monitored.

If training failed, it was impossible to determine whether

- the data term,
- physics term,
- KL divergence,

or another component was responsible.

### Code

```python
data_scale = ...
phys_scale = ...
state_reg_scale = ...

kl_and_prior = kl_value + eta_prior_scale * eta_prior_reg
```

The model now returns

- data loss
- physics loss
- KL (+ η prior)
- state regularization

separately.

### Explanation

Instead of seeing only a student's final exam grade, we now see:

- homework
- quizzes
- projects
- participation

This makes it much easier to diagnose what is going wrong during optimization.

---

## 5. Resume-Safe Freezing and Separate Learning Rates

### Objective

Ensure viscosity learns before the PINN begins updating.

### Problem

When training resumed from a checkpoint, the freeze schedule immediately ended because it depended on the absolute epoch number.

The PINN therefore started changing immediately.

### Code

```python
freeze_mean_net = epoch < freeze_until_epoch

set_module_requires_grad(
    raw_model.mean_net,
    not freeze_mean_net
)
```

Separate learning rates were also introduced:

```python
mean_net_lr
vgp_eta_lr
vgp_lambda_lr
```

### Explanation

The PINN is like an experienced student.

The VGP is the new student learning viscosity.

By freezing the PINN first, the experienced student cannot immediately answer every question.

The VGP is forced to learn.

After several hundred epochs, the PINN is allowed to make small adjustments using a much smaller learning rate.

---

## 6. Ice-Masked Farthest-Point Sampling

### Objective

Improve placement of Gaussian Process inducing points.

### Problem

Previously inducing points were placed on a simple rectangular grid.

Many points landed off the glacier where no useful information existed.

### Code

```python
placement = "ice_fps"
```

```python
selected.append(np.argmax(dists))
```

### Explanation

Instead of placing sensors uniformly across an empty map,

the new algorithm places them

- only on the glacier, and
- spreads them out as much as possible.

This allows the Gaussian Process to represent spatial variations in viscosity much more efficiently.

---

## 7. Continuous η Monitoring

### Objective

Detect viscosity collapse as early as possible.

### Problem

Previously the model could train for hours before it became obvious that η had failed.

### Code

```python
evaluate_eta_vs_reference(...)
```

Training now logs

- RMSE
- bias
- correlation

and issues warnings whenever

```text
log10_bias < -1
```

### Explanation

Instead of waiting until training finishes,

the model now periodically checks:

> "Am I actually learning viscosity?"

If not, training can be stopped early.

---

## 8. Gradient Diagnostics

### Objective

Verify that gradients actually reach η.

### Problem

Initially,

```text
grad_mean_net ≈ 100
grad_vgp_eta ≈ 1e-6
```

Nearly all optimization updated the PINN.

Very little updated viscosity.

### Code

```python
grad_norms = {
    "mean_net": ...,
    "vgp_eta": ...,
    "vgp_lambda": ...,
    "eta_log_shift": ...
}
```

Warnings are issued whenever

```text
mean_net >> vgp_eta
```

### Explanation

Gradients tell us which parameters are being updated.

These diagnostics answer:

> "Is training actually changing viscosity?"

If not, the optimizer configuration can be adjusted before wasting additional compute.

---

## 9. Training Stability Improvements

### Objective

Prevent auxiliary failures from stopping long-running jobs.

### Problem

Training occasionally crashed while writing plots or metrics, even though optimization itself was proceeding normally.

### Code

```python
if append:
    mode = "a"
else:
    mode = "w"
```

Additional safeguards include:

- saving checkpoints before plotting,
- catching plotting exceptions,
- preventing metric-writing issues from terminating training.

### Explanation

Previously, a plotting error could stop an entire multi-hour training run.

Now, visualization errors no longer interrupt optimization, making long-running cluster jobs much more robust.

---

## Training Progress

| Attempt | Main Changes | Result |
|----------|--------------|--------|
| Initial | Weak physics, equal learning rates | PINN absorbed errors; η ignored |
| Second | Strong physics, high η learning rate | η gradients improved, but viscosity collapsed toward zero |
| Current | η prior, state regularization, safer learning rate, moderate physics, ice-masked inducing points | Mean viscosity remains stable (bias ≈ −0.01 at epoch 391); next evaluation is after the PINN unfreezes (epoch ≥ 400) |

Overall, the debugging process progressively shifted optimization from modifying the glacier state variables to learning the latent viscosity field. While spatial correlation (`log10_r`) remains an area for improvement, the current configuration successfully prevents viscosity collapse and provides a much stronger foundation for physically meaningful inference.
