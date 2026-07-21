# Sequential VI Pipeline Validation

## Objective

Evaluate whether the **sequential PINN → VI** workflow improves recovery of the spatial viscosity field compared to the joint optimization approach.

## Results

Both stages of the sequential pipeline completed successfully.

### Performance Comparison

| Metric | Sequential VI-Only | Joint Training |
|--------|-------------------:|---------------:|
| `log10_eta_r` | **0.812** (best: **0.826**) | **0.744** |
| `log10_eta_rmse` | **0.278** | **0.292** |
| Mean Bias | ~**−0.15** | ~**−0.15** |

## Conclusions

The sequential VI-only pipeline achieves both **higher spatial correlation** and **lower RMSE** while maintaining a similar mean bias compared to the joint model.

These results provide strong evidence that the viscosity field is identifiable when the PINN is held fixed. The remaining limitation in the joint approach is therefore likely due to **interference from joint optimization**, rather than a fundamental identifiability issue.

## Notes

- Early stopping at **epoch 100** occurred as expected.
- The reported `inf`/`-inf` values for the λ and `rh` diagnostics are expected empty-range sentinel values because λ inference and continuity constraints are disabled in this experiment. They do **not** indicate numerical divergence or instability.
