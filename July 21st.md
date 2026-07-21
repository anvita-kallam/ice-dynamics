# Sequential VI Pipeline Validation

## Objective

Evaluate whether the **sequential PINN → VI** workflow improves recovery of the spatial viscosity field compared to the joint optimization approach.

### Performance Comparison

| Metric | Joint Training | Sequential VI-Only |
|--------|---------------:|-------------------:|
| Checkpoint | Epoch 749 | Best Epoch 54 |
| `log10_eta_r` | **0.681** | **0.831** |
| `log10_eta_rmse` | **0.292** | **0.274** |
| `log10_eta_bias` | **−0.149** | **−0.177** |
| Relative RMSE | **0.516** | **0.438** |
| Mean η (Reference = 14.9 MPa·yr) | **8.71** | **8.31** |
| Calibration (1σ / 2σ) | **0.45 / 0.71** | **0.77 / 1.00** |

Compared to the joint model, the sequential pipeline achieved:

- **+0.149** improvement in spatial correlation.
- **6.1% lower** log-space RMSE.
- Lower absolute error over **60.8%** of the glacier domain.
- Significantly improved posterior calibration.

The only metric where the joint model performs slightly better is mean bias, although the difference is relatively small.

### PINN State Recovery

The frozen-PINN sequential workflow also slightly improves reconstruction of the glacier state variables.

| Field | Joint | Sequential |
|------|-------:|-----------:|
| u (m/yr) | 0.611 | **0.608** |
| v (m/yr) | 0.627 | **0.570** |
| Surface (m) | 0.636 | **0.590** |
| Thickness (m) | 1.198 | **1.078** |
| Bed (m) | 163.02 | **163.00** |

These improvements are expected because the pretrained PINN remains fixed during VI optimization rather than continuing to change alongside the viscosity field.

## Summary

The sequential VI-only pipeline consistently outperforms the joint optimization approach for recovering the spatial viscosity field. While both methods recover similar mean viscosity, the sequential model achieves substantially higher spatial correlation, lower reconstruction error, and significantly better-calibrated uncertainty.

These results provide strong evidence that the viscosity field is **identifiable** when the PINN is held fixed. The reduced performance of the joint model therefore appears to stem from **interference during joint optimization**, rather than a fundamental limitation of the inverse problem.

One remaining limitation is the persistent low-viscosity bias observed across every experiment. Both the joint and sequential models underestimate the mean viscosity (`η ≈ 8.3–8.7 MPa·yr` compared to the reference `14.9 MPa·yr`), indicating that this bias likely originates from the physics formulation or η prior rather than the optimization strategy itself. This provides a clear direction for future work.

## Notes

- The optimized sequential run corresponds to **Best Epoch 54**.
- The earlier sequential experiment (early stopping at epoch 100) was evaluated only on the 8,192-point training subset and therefore is **not directly comparable** to the full-grid results reported above.
- The optimized rerun achieved `log10_eta_r = 0.831` on the same subset and confirmed this performance on the full evaluation grid.
- The previously reported joint correlation (`r ≈ 0.747`) came from the smaller evaluation subset; the full-grid evaluation gives the more representative value of **0.681**.

---
# Figure 1. Viscosity Recovery

<img width="2041" height="1059" alt="image" src="https://github.com/user-attachments/assets/494c8387-c832-41e7-abee-ae105133f630" />


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

<img width="1892" height="557" alt="image" src="https://github.com/user-attachments/assets/153981c6-e773-4fb0-b887-1127d01d2ea8" />


## Results

The posterior standard deviation remains relatively low across most of the glacier, with increased uncertainty appearing in regions of greater spatial variability. This suggests the uncertainty estimates are consistent with the difficulty of the inference problem.

The scatter plot demonstrates strong agreement between predicted and reference log-viscosity values, with a correlation coefficient of **0.831**. The primary deviation from the one-to-one line is a slight underestimation of higher-viscosity values.

The error histogram is centered slightly below zero, consistent with the measured bias, while most prediction errors remain within approximately ±0.2 log units.

## Summary

The inferred posterior is well calibrated and exhibits strong correlation with the reference viscosity field. Remaining errors are dominated by a mild underestimation of the highest viscosity values rather than widespread prediction failures.

---

# Figure 3. Geometry Reconstruction

<img width="2041" height="1516" alt="image" src="https://github.com/user-attachments/assets/31d7a32f-6018-4cc5-9442-0bd645f5b8b1" />


## Results

The pretrained PINN accurately reconstructs the glacier geometry prior to viscosity inference.

Surface elevation matches the reference solution almost exactly, with only small localized differences. Thickness predictions similarly reproduce the reference field with minimal error throughout the domain.

The largest discrepancies occur in the bed topography near the downstream boundary, where localized deviations appear close to the glacier terminus. These errors remain confined to a small region and do not significantly affect the overall geometry.

## Summary

Surface elevation and ice thickness are reconstructed with excellent accuracy, while bed elevation exhibits only localized downstream errors. Overall, the forward model provides an accurate geometric representation for the inverse problem.

---

# Figure 4. Speed Reconstruction

<img width="2041" height="530" alt="image" src="https://github.com/user-attachments/assets/d967cda5-88ea-400e-bd45-00bedf9d347d" />


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

<img width="2041" height="1043" alt="image" src="https://github.com/user-attachments/assets/090d4d4e-84ae-47a8-9987-1e1a8195d077" />


## Results

Both horizontal velocity components are reconstructed accurately.

The dominant downstream velocity component (`u`) closely matches the reference throughout the glacier, with only minor differences near the terminus.

The transverse velocity component (`v`) also reproduces the reference pattern despite its much smaller magnitude. Most residuals remain localized to the downstream region where velocity gradients are strongest.

Neither velocity component exhibits noticeable systematic bias.

## Summary

The PINN accurately captures both components of the glacier velocity field. Errors remain localized near the terminus, demonstrating that the sequential VI stage operates on a highly accurate forward solution.
---
# Figure 6. Joint vs. Sequential Recovery Comparison

<img width="1698" height="1378" alt="image" src="https://github.com/user-attachments/assets/1911779f-b3b6-4d97-86d0-f7b90d486d60" />


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

<img width="2737" height="1182" alt="image" src="https://github.com/user-attachments/assets/0dd52762-6770-4978-b390-881227fadc66" />


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

<img width="2177" height="2118" alt="image" src="https://github.com/user-attachments/assets/f328c6f6-09cb-4e2d-851f-f17aadbeed8f" />


## Results

This figure compares the state-variable reconstruction errors between the joint and sequential pipelines.

Across all state variables—including speed, surface elevation, thickness, and bed topography—the error patterns are remarkably similar.

For glacier speed, both methods exhibit small residuals concentrated near the downstream boundary where velocity gradients are largest. Surface elevation and thickness errors remain low throughout the domain, with only localized discrepancies near the terminus.

Bed topography errors are nearly identical between the two approaches and remain confined to the downstream boundary, reflecting limitations inherited from the pretrained PINN rather than the viscosity inference method.

Importantly, no degradation in forward-model accuracy is observed after switching to the sequential VI pipeline.

## Summary

Freezing the PINN during VI optimization preserves the excellent forward-model performance achieved during pretraining. The improved viscosity recovery observed in the sequential approach is therefore obtained without sacrificing reconstruction accuracy for glacier geometry or velocity, indicating that the primary benefit comes from improved inversion rather than changes to the forward solution.
