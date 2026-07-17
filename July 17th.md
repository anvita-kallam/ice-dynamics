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

## Figure 1: Joint Training Loss Components

<img width="1183" height="1221" alt="image" src="https://github.com/user-attachments/assets/c1d9c617-826a-4742-b0e5-0af0c99ff279" />

### Key Observations

- Training and validation ELBO closely overlap throughout all 1000 epochs.
- The ELBO decreases rapidly during the frozen PINN stage and decreases further after the PINN is unfrozen at **epoch 400**.
- Data loss remains stable while the PINN is frozen and gradually improves during joint fine-tuning.
- The KL (+ η prior) term quickly converges and remains stable, indicating that the η prior successfully prevents viscosity collapse.
- Physics loss remains nearly constant, while state regularization stays close to zero, showing that the pretrained glacier state is largely preserved.

**Takeaway:** Joint optimization remains stable throughout training and successfully fine-tunes the pretrained PINN without sacrificing physical consistency.

---

## Figure 2: Module Gradient Norms

<img width="1168" height="644" alt="image" src="https://github.com/user-attachments/assets/89c4822a-95c2-4e24-a699-f92904b6a6c3" />

### Key Observations

- During the first **400 epochs**, only the VGP receives gradients while the PINN is frozen.
- After unfreezing, the PINN gradients increase immediately and then gradually decrease as training converges.
- The VGP continues receiving gradients throughout training but remains much smaller than the PINN.

**Takeaway:** The staged optimization behaves as intended, but the PINN still dominates optimization after unfreezing, suggesting additional gradient balancing could further improve viscosity recovery.

---

## Figure 3: Overall Training Diagnostics

<img width="1200" height="1163" alt="image" src="https://github.com/user-attachments/assets/3d2771d7-1db5-46b5-9b33-439407ba6eb9" />

### Key Observations

- Stable optimization is maintained throughout the full training run.
- Training and validation remain closely aligned with no evidence of overfitting.
- The η prior successfully prevents the viscosity collapse observed in earlier experiments.
- The pretrained glacier state is preserved while allowing additional ELBO improvements after the PINN is unfrozen.

**Takeaway:** The revised optimization strategy provides stable training and successfully transitions from viscosity-only learning to joint fine-tuning.

---

## Validation Results

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
## Figure 1: Viscosity Recovery

<img width="2041" height="1054" alt="image" src="https://github.com/user-attachments/assets/ccb17bf1-20ba-475e-9fd2-3d7682e8e334" />


### Analysis

- The predicted viscosity field is nearly uniform, whereas the ground truth contains significant spatial variability.
- The mean viscosity is recovered accurately (`log10` bias ≈ **−0.008**, RMSE ≈ **0.287**), confirming that the η-prior successfully prevents viscosity collapse.
- However, the difference maps and scatter plot show almost no relationship between predicted and true spatial patterns (`r ≈ 0.002`).
- The VI uncertainty is nearly constant across the domain, indicating that the GP is not identifying regions of higher uncertainty.

<img width="1892" height="557" alt="image" src="https://github.com/user-attachments/assets/460e913e-4fb2-4f81-82ab-ae5623e54c8e" />


**Takeaway:** The model accurately predicts the average viscosity but fails to recover the spatial distribution, producing an almost constant viscosity field.

---

## Figure 2: Geometry Prediction

<img width="2041" height="1516" alt="image" src="https://github.com/user-attachments/assets/7dfb6255-f287-4863-b38d-1ad235119d62" />


### Analysis

- The PINN accurately reconstructs the glacier surface and thickness, with only small localized errors.
- Most prediction errors occur near the glacier terminus, where gradients are largest.
- Bed elevation shows the largest discrepancy, particularly near the downstream boundary.

**Takeaway:** The pretrained PINN preserves glacier geometry well, indicating that geometry prediction is not limiting viscosity inference.

---

## Figure 3: Speed Prediction

<img width="2041" height="531" alt="image" src="https://github.com/user-attachments/assets/fa6ad95d-6631-4e9a-b8ca-45e96ecdd4de" />



### Analysis

- The predicted speed field closely matches the ground truth with a relative RMSE of approximately **2.5%**.
- Differences are small over most of the glacier and are concentrated near the terminus where velocities are highest.
- The overall flow dynamics are accurately reproduced.

**Takeaway:** The PINN maintains excellent velocity predictions even after joint training, confirming that introducing VI does not significantly degrade the forward solution.

---

## Figure 4: Velocity Components

<img width="2041" height="1033" alt="image" src="https://github.com/user-attachments/assets/74dde1af-e94d-429a-91c2-87ecfc525c80" />


### Analysis

- Both horizontal velocity components (`u` and `v`) closely match the reference solution.
- Errors remain small across most of the domain and are primarily concentrated near the terminus and lateral boundaries.
- The transverse velocity (`v`) captures the expected flow structure with only minor localized differences.

**Takeaway:**  Joint optimization preserves both velocity components with high accuracy, demonstrating that the revised training strategy maintains the quality of the pretrained PINN while learning viscosity.

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
