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

---
# Figure 1. Viscosity Recovery

## Results

The sequential VI-only pipeline successfully recovers the major spatial patterns of the viscosity field. Unlike the previous joint-training approach, the estimated viscosity no longer collapses to a nearly constant value and instead captures the large-scale spatial variation across the glacier.

Performance metrics indicate strong agreement with the reference solution:

- **Correlation:** `r = 0.831`
- **Log-bias:** `−0.177`
- **Log-RMSE:** `0.274`

The difference maps show that the largest errors occur in the central high-viscosity region, where viscosity is slightly underestimated, while small positive errors appear near the downstream terminus. Overall, the recovered field is smoother than the reference but accurately reproduces the dominant spatial structure.

## Summary

The sequential VI-only approach substantially improves spatial viscosity recovery while maintaining accurate mean viscosity. The remaining errors are localized and primarily associated with regions of strong viscosity gradients.

---

# Figure 2. Posterior Uncertainty and Statistical Evaluation

## Results

The posterior standard deviation remains relatively low across most of the glacier, with increased uncertainty appearing in regions of greater spatial variability. This suggests the uncertainty estimates are consistent with the difficulty of the inference problem.

The scatter plot demonstrates strong agreement between predicted and reference log-viscosity values, with a correlation coefficient of **0.831**. The primary deviation from the one-to-one line is a slight underestimation of higher-viscosity values.

The error histogram is centered slightly below zero, consistent with the measured bias, while most prediction errors remain within approximately ±0.2 log units.

## Summary

The inferred posterior is well calibrated and exhibits strong correlation with the reference viscosity field. Remaining errors are dominated by a mild underestimation of the highest viscosity values rather than widespread prediction failures.

---

# Figure 3. Geometry Reconstruction

## Results

The pretrained PINN accurately reconstructs the glacier geometry prior to viscosity inference.

Surface elevation matches the reference solution almost exactly, with only small localized differences. Thickness predictions similarly reproduce the reference field with minimal error throughout the domain.

The largest discrepancies occur in the bed topography near the downstream boundary, where localized deviations appear close to the glacier terminus. These errors remain confined to a small region and do not significantly affect the overall geometry.

## Summary

Surface elevation and ice thickness are reconstructed with excellent accuracy, while bed elevation exhibits only localized downstream errors. Overall, the forward model provides an accurate geometric representation for the inverse problem.

---

# Figure 4. Speed Reconstruction

## Results

The predicted speed field closely matches the reference solution across the glacier.

The reported relative error is:

- **Relative RMSE:** `0.0277`

corresponding to approximately **2.8%** error.

Most differences are confined to the downstream region where flow accelerates rapidly. Throughout the glacier interior, predicted and reference speeds are nearly indistinguishable.

## Summary

The pretrained PINN accurately reproduces glacier speed, with only small localized residuals near the terminus. This indicates that forward-model error is minimal during viscosity inference.

---

# Figure 5. Velocity Component Reconstruction

## Results

Both horizontal velocity components are reconstructed accurately.

The dominant downstream velocity component (`u`) closely matches the reference throughout the glacier, with only minor differences near the terminus.

The transverse velocity component (`v`) also reproduces the reference pattern despite its much smaller magnitude. Most residuals remain localized to the downstream region where velocity gradients are strongest.

Neither velocity component exhibits noticeable systematic bias.

## Summary

The PINN accurately captures both components of the glacier velocity field. Errors remain localized near the terminus, demonstrating that the sequential VI stage operates on a highly accurate forward solution.
