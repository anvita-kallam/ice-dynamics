# Model Hyperparameter Tuning to Improve Joint PINN + VI Training

## Objective
The current joint PINN + Variational Inference implementation successfully minimizes the total ELBO, but it appears to be doing so by modifying the PINN state variables (velocity and thickness) instead of correctly inferring the latent viscosity field (η). The resulting viscosity predictions exhibit a strong systematic bias (approximately one order of magnitude too low) and essentially zero correlation with the reference viscosity field, despite stable optimization.

I want to modify the training procedure so that the optimizer is encouraged (or forced) to explain the observations through changes in viscosity rather than changes in velocity or thickness.

### Current Training Pipeline

Current workflow:

1. Pretrain PINN on Icepack observations (velocity, thickness, etc.).
2. Jointly optimize the pretrained PINN together with the Variational Inference model.
3. Total loss is

$$
L = L_{\text{data}} + L_{\text{PDE}} + L_{\text{KL}}
$$

where

- **Data loss:** difference between PINN predictions and Icepack outputs.
- **PDE loss:** momentum / governing equation residual.
- **KL loss:** variational regularization.

Currently, the optimizer is free to reduce the loss by changing either

- viscosity (desired), or
- velocity/thickness fields (undesired).

The optimization seems to choose the easier path of modifying the glacier state variables rather than recovering the correct viscosity.

## Why the Optimizer Preferred Velocity/Thickness Over η

The previous **more_sliding** joint training run minimized the ELBO primarily by modifying the PINN state variables rather than recovering the latent viscosity field. Several factors contributed to this behavior:

### Root Causes:

### 1. Freeze Was Ineffective on Resume

The freeze schedule used the absolute epoch number:

```python
epoch < 28
```

When training resumed from **epoch 724**, the condition was immediately false, causing the pretrained `mean_net` to unfreeze from the very beginning of the resumed run.

---

### 2. Equal Learning Rates

Both networks used the same learning rate:

```text
mean_net = 2e-4
vgp_eta  = 2e-4
```

Because the PINN contains many more trainable parameters than the sparse Gaussian Process for η, it was much easier for optimization to reduce the loss by modifying velocity and thickness than by adjusting viscosity.

---

### 3. Physics Loss Was Too Weak

The configuration

```text
ssa_*_std = 5
phys_scale = 0.1
```

made the physics term extremely permissive.

The SSA residuals were much smaller than the assumed standard deviation, so the PDE likelihood remained near its Gaussian constant floor (~0.506), producing almost no gradient with respect to η.

As a result, the physics loss contributed little information for recovering viscosity.

---

### 4. No Constraint to Preserve the Pretrained PINN

Once the PINN was unfrozen, nothing encouraged

$$
(u, v, s, H)
$$

to remain close to the pretrained Stage-1 solution.

The optimizer therefore altered the glacier state variables to reduce the ELBO instead of modifying viscosity.

---

### 5. Gradient Imbalance

Gradient logging showed a dramatic imbalance:

```text
grad_mean_net ≈ 1e2
grad_vgp_eta  ≈ 1e-6
```

Nearly all optimization effort flowed into the PINN while the viscosity parameters received almost no updates.

Additionally, λ remained part of the ELBO despite not being used in the SSA formulation.

---

## Implemented Improvements

| Change | Purpose |
|---------|---------|
| Resume-safe freeze | Freeze duration is now counted relative to `start_epoch`, ensuring resumed runs still honor the freeze period. |
| State regularization | Added a soft MSE penalty between current PINN outputs and the frozen Stage-1 reference (`state_reg_scale`) to discourage unnecessary changes to velocity, thickness, and geometry. |
| Configurable loss weights | Added configurable `data_scale`, `phys_scale`, and `state_reg_scale`. |
| Separate learning rates | Independent learning rates for `mean_net`, `vgp_eta`, and `vgp_lambda`. |
| Learning-rate scheduler | Added cosine decay and ReduceLROnPlateau options. |
| Richer logging | Log individual loss terms (data, physics, KL, state regularization, total), per-module gradient norms, learning rates, and η metrics (RMSE, bias, correlation) every `eta_eval_every` epochs. |
| Stronger physics constraints | Reduced `ssa_*_std` and disabled unnecessary λ regularization (`kl_lambda = 0`) so the physics term contributes meaningful gradients for viscosity recovery. |

---

## Updated Training Configuration

The current `run_torch.cfg` is configured for an **η-first joint training strategy**:

- `restore = False`
- Freeze the PINN for the first **400 epochs**
- `mean_net_lr << vgp_eta_lr`
- `state_reg_scale = 1`
- `phys_scale = 1`
- `ssa_*_std = 1`

This setup encourages the optimizer to first recover viscosity before allowing the PINN to make small adjustments.

---

## Next Experiment (Cluster)

```bash
# Sync Archive/ to DSI, then:
cd ~/ice-dynamics/Archive

# Optional: move old joint checkpoint so training starts fresh
mv checkpoints/torch_joint/more_sliding \
   checkpoints/torch_joint/more_sliding_prev_biased

# Submit new training job
sbatch slurm/vi_train_more_sliding.sbatch
```

---

## Expected Behavior During Training

Monitor the logs for the following indicators:

### Early Training (Frozen PINN)

- `mean_net` remains frozen.
- `grad_vgp_eta` increases significantly.
- η parameters receive the majority of the optimization updates.

### Viscosity Recovery

Watch for:

```text
eta_vs_ref ...
```

Expected improvements:

- `log10_bias` moves toward **0** (previously approximately **−1.06**).
- `log10_r` becomes positive and steadily increases.
- RMSE decreases over time.

### After Epoch 400

Once the freeze period ends:

- `mean_net` unfreezes.
- It trains using a much smaller learning rate than η.
- `state_reg` should remain small, indicating the pretrained glacier state is being preserved while viscosity continues to improve.
