# Final Joint Training Results

## Objective

Evaluate whether the revised joint PINN + Variational Inference training strategy successfully learns the latent viscosity field while maintaining stable optimization and preserving the pretrained glacier state.

---

## Training Summary

The joint training completed successfully after **1000 epochs (~8 hours)** with no optimization failures or viscosity collapse.

| Metric | Final Value |
|---------|------------:|
| Final epoch | **999** |
| `log10_bias` | **−0.012** |
| Mean predicted η | **11.95 MPa·yr** (reference: **15.04**) |
| `log10_r` | **~0.15** |
| Gradient ratio (`mean_net : vgp_eta`) | **~6000×** |

Overall, the revised training strategy achieved stable optimization and nearly unbiased recovery of the **mean viscosity**, but recovering the **spatial viscosity distribution** remains the primary challenge.

---

# Figure 1: Joint Training Loss Components

<img width="1183" height="1221" alt="image" src="https://github.com/user-attachments/assets/c1d9c617-826a-4742-b0e5-0af0c99ff279" />

### Key Observations

- Training and validation ELBO closely overlap throughout all 1000 epochs.
- The ELBO decreases rapidly during the frozen PINN stage and decreases further after the PINN is unfrozen at **epoch 400**.
- Data loss remains stable while the PINN is frozen and gradually improves during joint fine-tuning.
- The KL (+ η prior) term quickly converges and remains stable, indicating that the η prior successfully prevents viscosity collapse.
- Physics loss remains nearly constant, while state regularization stays close to zero, showing that the pretrained glacier state is largely preserved.

**Takeaway:** Joint optimization remains stable throughout training and successfully fine-tunes the pretrained PINN without sacrificing physical consistency.

---

# Figure 2: Module Gradient Norms

<img width="1168" height="644" alt="image" src="https://github.com/user-attachments/assets/89c4822a-95c2-4e24-a699-f92904b6a6c3" />

### Key Observations

- During the first **400 epochs**, only the VGP receives gradients while the PINN is frozen.
- After unfreezing, the PINN gradients increase immediately and then gradually decrease as training converges.
- The VGP continues receiving gradients throughout training but remains much smaller than the PINN.

**Takeaway:** The staged optimization behaves as intended, but the PINN still dominates optimization after unfreezing, suggesting additional gradient balancing could further improve viscosity recovery.

---

# Figure 3: Overall Training Diagnostics

<img width="1200" height="1163" alt="image" src="https://github.com/user-attachments/assets/3d2771d7-1db5-46b5-9b33-439407ba6eb9" />

### Key Observations

- Stable optimization is maintained throughout the full training run.
- Training and validation remain closely aligned with no evidence of overfitting.
- The η prior successfully prevents the viscosity collapse observed in earlier experiments.
- The pretrained glacier state is preserved while allowing additional ELBO improvements after the PINN is unfrozen.

**Takeaway:** The revised optimization strategy provides stable training and successfully transitions from viscosity-only learning to joint fine-tuning.

---

# Validation Results

| Metric | Previous VI | η-Prior Run |
|---------|------------:|------------:|
| `log10_eta_bias` | ~−1.06 | **−0.008** |
| `log10_eta_rmse` | ~1.11 | **0.287** |
| `log10_eta_r` | ~−0.03 | **~0.00** |
| Mean predicted η | ~0.6 | **~12** |
| Mean reference η | ~15 | **~15** |
| Speed relative RMSE | ~0.02 | **0.025** |

### Interpretation

The η-prior strategy successfully eliminated the primary failure mode from previous experiments.

**Improvements**
- Eliminated viscosity collapse.
- Reduced viscosity bias from **~−1.06** to **−0.008**.
- Reduced viscosity RMSE from **~1.11** to **0.287**.
- Preserved PINN prediction accuracy with only a minor increase in speed RMSE.

**Remaining Limitation**

While the model accurately predicts the **mean viscosity**, it still fails to recover meaningful **spatial variations** (`log10_eta_r ≈ 0`). The VGP largely predicts a nearly constant viscosity field centered near the prior rather than a spatially varying viscosity map.

---

# Conclusions

The revised optimization strategy successfully addresses the stability issues encountered during earlier training runs.

Key achievements include:

- Stable optimization over the full 1000 epochs.
- No viscosity collapse.
- Nearly unbiased mean viscosity estimates.
- Preservation of the pretrained glacier state.
- Consistent train/validation behavior with no overfitting.

The primary remaining challenge is improving the **spatial recovery** of viscosity while maintaining the strong mean accuracy achieved by the current model.

---

# Next Steps

Future work will focus on improving the spatial reconstruction of η by:

- Generating posterior η maps for visual evaluation.
- Reducing the strength of the η prior.
- Increasing the influence of the physics loss.
- Increasing the number of VGP inducing points.
- Extending the frozen PINN stage before joint optimization.

The goal is to improve spatial correlation while preserving the stable and unbiased mean viscosity recovered by the current training strategy.
