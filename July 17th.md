# Final Training Summary

## Objective

Complete the full joint PINN + Variational Inference training run and evaluate whether the revised optimization strategy successfully learns the latent viscosity field while maintaining stable training.

---

## Training Outcome

The joint training completed successfully.

- **Training duration:** ~8 hours
- **Final epoch:** 999
- **Exit status:** 0 (successful completion)

---

## Final Metrics

| Metric | Final Value | Interpretation |
|---------|------------|----------------|
| `log10_bias` | **−0.012** | Nearly unbiased mean viscosity prediction. |
| Mean predicted η | **11.95 MPa·yr** | Close to the reference mean of **15.04 MPa·yr**. |
| `log10_r` | **~0.15** | Weak recovery of the spatial viscosity distribution. |
| Gradient ratio (`mean_net : vgp_eta`) | **~6000×** | The PINN still receives much larger gradients than the viscosity GP during joint optimization. |

---

## Interpretation

The final training run demonstrates that the revised optimization strategy successfully solved the primary stability issues encountered during earlier experiments.

Positive outcomes include:

- Stable optimization throughout all 1000 epochs.
- No viscosity collapse.
- Nearly unbiased recovery of the average viscosity.
- Consistent convergence with successful completion of the full training schedule.

However, the diagnostics indicate that recovering the **spatial structure** of the viscosity field remains challenging.

Although the average viscosity is estimated accurately, the relatively low correlation (`log10_r ≈ 0.15`) suggests that the model still struggles to reproduce fine-scale spatial variations. Additionally, the large gradient imbalance (approximately **6000×** larger gradients for the PINN than the VGP) indicates that the PINN continues to dominate optimization during joint training, leaving room for further improvements in directing learning toward η.

Next I will generate posterior predictions using the trained model and update the diagnostic figures.

```bash
cd ~/ice-dynamics/Archive

sbatch slurm/vi_predict_more_sliding.sbatch
```

The prediction stage will generate posterior viscosity estimates and refreshed evaluation plots for further analysis of the recovered viscosity field.

---

# Figure 1: Joint Training Loss Components

<img width="1183" height="1221" alt="image" src="https://github.com/user-attachments/assets/c1d9c617-826a-4742-b0e5-0af0c99ff279" />


## Total ELBO

### Observations

- Training and validation ELBO overlap almost perfectly throughout all 1000 epochs.
- The ELBO decreases rapidly during the first ~50 epochs, then remains stable while the PINN is frozen.
- At **epoch 400**, unfreezing the PINN results in a second gradual decrease before converging around epochs 800–900.
- No divergence between training and validation is observed.

### Interpretation

The optimization remains stable throughout training. The additional reduction in ELBO after epoch 400 shows that jointly optimizing the PINN and VGP improves the solution without introducing instability or overfitting.

---

## Data Loss and KL / η Prior

### Observations

- Data loss remains nearly constant during the frozen stage.
- After the PINN is unfrozen, the data loss gradually decreases before stabilizing near the end of training.
- The KL (+ η prior) term decreases rapidly during early training and then remains nearly constant, with only a few isolated spikes.

### Interpretation

Most viscosity learning occurs while the PINN is frozen. After unfreezing, the optimizer primarily fine-tunes the pretrained PINN while maintaining the learned viscosity. The stable KL term indicates that the η prior successfully prevents viscosity collapse.

---

## Physics Loss and State Regularization

### Observations

- Physics NLL remains essentially constant throughout training.
- State regularization remains close to zero across all epochs.

### Interpretation

The governing equations remain satisfied throughout optimization, and the pretrained glacier state changes very little. This indicates that the optimizer is refining viscosity while preserving the pretrained PINN solution.

---

# Figure 2: Module Gradient Norms

<img width="1168" height="644" alt="image" src="https://github.com/user-attachments/assets/89c4822a-95c2-4e24-a699-f92904b6a6c3" />


## Observations

- During the first **400 epochs**, only the VGP receives gradients because the PINN is frozen.
- At epoch 400, the PINN is unfrozen and its gradient norm increases immediately.
- The PINN gradients gradually decrease as optimization converges.
- The VGP continues receiving gradients throughout training, although they remain much smaller than the PINN gradients.

## Interpretation

The staged optimization behaves as intended by allowing viscosity to learn before joint optimization begins. However, once the PINN is unfrozen, it again dominates the optimization, suggesting that additional gradient balancing may further improve recovery of the viscosity field.

---

# Figure 3: Overall Training Summary

<img width="1200" height="1163" alt="image" src="https://github.com/user-attachments/assets/3d2771d7-1db5-46b5-9b33-439407ba6eb9" />


## Key Takeaways

- Stable optimization was maintained throughout the full **1000-epoch** training run.
- Training and validation losses remain closely aligned, indicating no evidence of overfitting.
- The staged training strategy successfully transitions from frozen PINN optimization to joint fine-tuning.
- The η prior and state regularization prevent the viscosity collapse observed in earlier experiments.
- The pretrained glacier state is preserved while still allowing improvements to the ELBO after unfreezing.
- The primary remaining limitation is that the PINN continues to receive much larger gradients than the VGP after epoch 400, likely contributing to the weaker recovery of the spatial viscosity distribution.
---

# Validation Results

| Metric | Previous VI Run | η-Prior Run (Epoch 999) |
|---------|-----------------|-------------------------|
| `log10_eta_bias` | ~−1.06 | **−0.008** |
| `log10_eta_rmse` | ~1.11 | **0.287** |
| `log10_eta_r` | ~−0.03 | **~0.00** |
| Mean predicted η | ~0.6 | **~12** |
| Mean reference η | ~15 | **~15** |
| Speed relative RMSE | ~0.02 | **0.025** |

---

## Interpretation

The η-prior strategy successfully resolved the major failure mode observed in previous runs.

### Improvements

- The viscosity collapse was eliminated.
- The mean viscosity prediction is now nearly unbiased (`log10_eta_bias ≈ -0.008`).
- The viscosity RMSE improved substantially (from **~1.11** to **0.287**).
- The pretrained PINN maintained good predictive performance, with only a small increase in speed RMSE.

### Remaining Limitation

Although the model accurately recovers the **average viscosity**, it still fails to recover the **spatial distribution** of viscosity.

The correlation (`log10_eta_r ≈ 0`) indicates that the VGP is predicting an almost constant viscosity field centered near the prior (`η_init ≈ 12`) rather than learning meaningful spatial variations across the glacier.

Overall, the revised optimization strategy successfully fixes the η→0 collapse and preserves the physical solution, but further improvements are needed to recover spatially varying viscosity.

---

# Next Steps

Future work should focus on improving the spatial recovery of η rather than its overall magnitude. Possible directions include:

- Refresh the final training plots using the completed training logs.
- Generate posterior η maps from the HDF5 outputs to visually evaluate the recovered viscosity field.
- Experiment with an η-focused training recipe by:
  - reducing the strength of the η prior,
  - increasing the influence of the physics loss,
  - increasing the number of VGP inducing points, and
  - keeping the PINN frozen for a longer portion of training.

The primary objective of future tuning is to improve the spatial correlation (`log10_eta_r`) while maintaining the unbiased mean viscosity achieved by the current model.
