# Evaluate Sequential Model Suite

## Objective

Compare the performance of multiple optimized **sequential (VI-only)** training configurations to identify the best approach for improving viscosity recovery while maintaining high spatial accuracy.

### Experimental Design

Eight controlled experiments were launched, each modifying **one primary component** of the baseline sequential pipeline (unless otherwise noted). All runs were initialized with the same random seed (**20260721**) to ensure differences in performance could be attributed to the configuration changes rather than initialization variability.

| Configuration | Controlled Change | Purpose |
|---------------|-------------------|---------|
| **control_adamw** | Fresh rerun of the current best sequential model | Establish a reproducible baseline for comparison. |
| **weak_prior** | η prior scale **0.08 → 0.03**, prior std **1.5 → 2.5**, KL **0.15 → 0.10** | Reduce prior influence so observations and physics have greater impact on the inferred viscosity field. |
| **raised_prior_center** | Prior mean **12 → 15 MPa·yr** | Shift the prior toward the expected physical viscosity to reduce the systematic low-viscosity bias observed in previous experiments. |
| **strong_physics** | Physics weight **4 → 6**, SSA observation std **0.06 → 0.05** | Increase the influence of the governing physics equations during optimization. |
| **gp_capacity** | **32×32** inducing points, **Matérn-5/2** kernel, anisotropic length scales | Increase Gaussian Process expressiveness to better capture fine spatial variations in viscosity. |
| **optimizer_ngd** | AdamW + Natural Gradients | Evaluate whether a more principled optimizer improves posterior convergence. |
| **optimizer_fast_cosine** | Learning rate **5e-4** with a shorter cosine schedule | Encourage faster convergence while maintaining training stability. |
| **combined_candidate** | Raised prior center + weak prior/KL + stronger physics + higher GP capacity | Combine the most promising modifications into a single candidate production configuration. |

### Completed Runs

| Job ID | Configuration | Status | Wall Time |
|--------:|---------------|--------|----------:|
| 1191131 | control_adamw | Completed | 5h 04m |
| 1191132 | weak_prior | Completed | 5h 54m |
| 1191133 | raised_prior_center | Completed | 8h 03m |

### Runs Still in Progress

Approximately **43 minutes** remained before wall time.

| Configuration | Best Correlation (r) | Latest | Mean η | Current Assessment |
|---------------|---------------------:|--------|--------:|-------------------|
| strong_physics | **0.821** | 0.818 (Epoch 160) | 7.87 | Meets correlation target; mean viscosity remains low. |
| optimizer_fast_cosine | **0.832** | 0.808 (Epoch 104) | 8.81 | Strong early performance; mean viscosity similar to baseline. |
| gp_capacity | **0.818** | 0.796 (Epoch 122) | 8.55 | Improved flexibility but currently below target correlation. |
| optimizer_ngd | **0.739** | 0.739 (Epoch 92) | 8.63 | Underperforming; unlikely to outperform the baseline. |
| combined_candidate | **0.719** | 0.719 (Epoch 87) | 4.06 | Combined modifications appear to over-constrain optimization and significantly reduce performance. |

### Additional Experiment

Among the completed runs, **raised_prior_center** produced the most promising improvement by increasing the inferred mean viscosity while maintaining strong spatial recovery. Based on these preliminary results, an additional follow-up experiment was submitted with the **prior center increased further from 15 MPa·yr to 18 MPa·yr** to evaluate whether the remaining systematic low-viscosity bias can be reduced without degrading spatial correlation.

### Next Steps

- Complete all remaining sequential training runs.
- Compare every configuration using:
  - Spatial correlation
  - RMSE
  - Mean viscosity
  - Bias
  - Posterior calibration
- Evaluate the new **18 MPa·yr prior-center** experiment.
- Select the best-performing sequential configuration for comparison against the joint optimization pipeline and adopt it as the new default training recipe.
