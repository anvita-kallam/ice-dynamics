# Evaluate Sequential Model Suite

## Objective

Compare the performance of multiple optimized **sequential (VI-only)** training configurations to identify the best approach for improving viscosity recovery while maintaining high spatial accuracy.

### Completed Runs

| Job ID | Configuration | Status | Wall Time |
|--------:|---------------|--------|----------:|
| 1191131 | control_adamw | Completed | 5h 04m |
| 1191132 | weak_prior | Completed | 5h 54m |
| 1191133 | raised_prior_center | Completed | 8h 03m |

### Runs Still in Progress

Approximately **43 minutes** remaining before wall time.

| Configuration | Best Correlation (r) | Latest | Mean η | Current Assessment |
|---------------|---------------------:|--------|--------:|-------------------|
| strong_physics | **0.821** | 0.818 (Epoch 160) | 7.87 | Meets correlation target; mean viscosity remains low |
| fast_cosine | **0.832** | 0.808 (Epoch 104) | 8.81 | Strong early performance; mean viscosity similar to control |
| gp_capacity | **0.818** | 0.796 (Epoch 122) | 8.55 | Below target correlation so far |
| NGD | **0.739** | 0.739 (Epoch 92) | 8.63 | Underperforming; unlikely to be competitive |
| combined | **0.719** | 0.719 (Epoch 87) | 4.06 | Significantly underperforming |

### Next Steps

- Complete all remaining sequential training runs.
- Compare every configuration using:
  - Spatial correlation
  - RMSE
  - Mean viscosity
  - Bias
  - Calibration
- Select the best-performing sequential configuration for further analysis and comparison against the joint optimization pipeline.
