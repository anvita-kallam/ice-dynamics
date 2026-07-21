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
---
# Figure 6. Joint vs. Sequential Recovery Comparison

## Results

This figure directly compares the posterior viscosity recovery obtained using the joint optimization pipeline and the sequential (frozen-PINN) VI pipeline.

The scatter plot shows that both approaches reproduce the general relationship between the true and inferred viscosity, but the sequential model follows the one-to-one trend much more closely across the full viscosity range. The joint model exhibits stronger compression toward the mean, particularly for higher-viscosity regions.

The error distributions further illustrate this improvement. The sequential model produces a narrower distribution with fewer large positive errors, while the joint model exhibits a broader tail, indicating larger overall reconstruction errors.

The uncertainty-versus-error comparison shows that the sequential model maintains higher posterior uncertainty while simultaneously achieving lower reconstruction error. In contrast, the joint model is relatively overconfident despite producing larger errors, suggesting poorer uncertainty calibration.

The recovery summary confirms these trends quantitatively:

- **Lower RMSE:** Sequential (0.274) vs. Joint (0.292)
- **Higher spatial correlation:** Sequential (`r = 0.831`) vs. Joint (`r ≈ 0.68`)
- **Comparable bias:** Both models retain similar mean bias, with the joint model exhibiting a slightly smaller absolute bias.

## Summary

The sequential VI-only approach consistently outperforms the joint optimization strategy in recovering the spatial viscosity field. Although both methods recover similar mean viscosity, freezing the PINN allows the VGP to infer substantially more accurate spatial structure while maintaining well-calibrated uncertainty estimates.

---

# Figure 7. Spatial Comparison of Joint and Sequential Recovery

## Results

This figure compares the recovered viscosity fields produced by the joint and sequential pipelines.

Both methods recover the broad spatial distribution of viscosity, but the sequential solution preserves noticeably stronger spatial contrast. The joint solution remains more spatially uniform, particularly across the central high-viscosity region.

The difference maps show that the sequential model consistently predicts slightly higher viscosities throughout the central glacier while reducing the excessive smoothing present in the joint solution.

The absolute-error reduction map demonstrates that the sequential model lowers reconstruction error across much of the glacier. Improvements are especially evident in the interior high-viscosity region, while only a few localized regions show slightly larger errors than the joint approach.

The final binary comparison confirms this trend, with most pixels indicating lower absolute error for the sequential solution.

## Summary

The sequential pipeline produces a more spatially accurate viscosity field across most of the glacier. Improvements are widespread rather than localized, indicating that freezing the PINN substantially reduces the oversmoothing introduced during joint optimization.

---

# Figure 8. State Recovery Comparison

## Results

This figure compares the state-variable reconstruction errors between the joint and sequential pipelines.

Across all state variables—including speed, surface elevation, thickness, and bed topography—the error patterns are remarkably similar.

For glacier speed, both methods exhibit small residuals concentrated near the downstream boundary where velocity gradients are largest. Surface elevation and thickness errors remain low throughout the domain, with only localized discrepancies near the terminus.

Bed topography errors are nearly identical between the two approaches and remain confined to the downstream boundary, reflecting limitations inherited from the pretrained PINN rather than the viscosity inference method.

Importantly, no degradation in forward-model accuracy is observed after switching to the sequential VI pipeline.

## Summary

Freezing the PINN during VI optimization preserves the excellent forward-model performance achieved during pretraining. The improved viscosity recovery observed in the sequential approach is therefore obtained without sacrificing reconstruction accuracy for glacier geometry or velocity, indicating that the primary benefit comes from improved inversion rather than changes to the forward solution.
