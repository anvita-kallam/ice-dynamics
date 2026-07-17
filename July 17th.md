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

---

## Next Step

Generate posterior predictions using the trained model and update the diagnostic figures.

```bash
cd ~/ice-dynamics/Archive

sbatch slurm/vi_predict_more_sliding.sbatch
```

The prediction stage will generate posterior viscosity estimates and refreshed evaluation plots for further analysis of the recovered viscosity field.
