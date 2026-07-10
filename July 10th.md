# Run Prediction + Start Plotting Model Training Performance

## Objective

- Run the prediction stage using the completed VI model to generate posterior viscosity estimates.
- Begin analyzing model performance by plotting training and validation losses over time.

## Prediction

- Submitted the prediction job using the trained VI checkpoint.
- Generated posterior predictions for the synthetic test case for comparison with the Icepack ground truth.


### Pipeline Status

| Stage | Status |
|--------|--------|
| Pretrain | ✅ Completed |
| VI Training | ✅ Completed (Job 978514) |
| Prediction | ✅ Completed |

### Initial Plotting

<img width="1550" height="944" alt="image" src="https://github.com/user-attachments/assets/95de464b-e2b4-49ac-b080-2fb8c77bcc5b" />

The plot itself isn't evidence that training is bad—it mainly indicates that the visualization is masking most of the optimization because the initial loss is orders of magnitude larger than the final loss. However, it also raises the possibility that the PINN reached its useful minimum very early, which is exactly why the earlier note ("plot loss from VI to see if it saturated too soon") is important. A log-scale plot and separate loss components will tell me whether the model truly converged or whether the current plot is simply hiding meaningful progress.

## PINN Pretraining Loss Analysis

<img width="842" height="1142" alt="image" src="https://github.com/user-attachments/assets/17ac571c-f1d6-4ef1-b110-848de5fb3161" />

The updated log-scale training curves indicate that the PINN pretraining process is behaving as expected. The original linear-scale plot was misleading because the initial loss dominated the y-axis, making later optimization appear flat. On a log scale, the model continues making steady progress throughout training.

---

### Key Observations:

### 1. Continuous Optimization

Rather than converging after the first few epochs, the model continues improving over the full 1000 epochs.

- **Epochs 0–50:** Rapid reduction in loss as the network learns the coarse structure.
- **Epochs 50–300:** Steady improvement with a slower optimization rate.
- **Epochs 300–1000:** Continued gradual decrease, indicating the optimizer is still refining the solution.

This behavior is typical for neural network optimization and suggests the model has not become completely stagnant.

---

### 2. Training vs. Validation Loss

The validation loss closely follows the training loss throughout optimization.

**Positive signs:**

- No noticeable overfitting.
- No divergence between train and validation curves.
- Validation performance continues improving alongside training.

The occasional validation spikes are small and likely caused by stochastic optimization or mini-batch sampling rather than instability.

---

### 3. Field Component Behavior

The individual prediction targets exhibit different convergence characteristics.

#### u (x-velocity)

- Highest noise throughout training.
- Converges more slowly than the other fields.
- May have a larger dynamic range or more difficult prediction task.
- Worth verifying that normalization matches the other variables.

#### v (y-velocity)

- Smooth optimization.
- Stable convergence.
- No obvious optimization issues.

#### s (Surface Elevation)

- Fastest convergence.
- Achieves the lowest final loss.
- Appears to be the easiest field for the network to learn.

#### h (Ice Thickness)

- Smooth optimization.
- Similar behavior to surface elevation.
- Slightly slower convergence than surface elevation.

---

## Training Efficiency

Around **epochs 300–400**, the optimization noticeably slows.

Although the loss continues decreasing, the rate of improvement becomes much smaller.

Possible next experiments:

- Train for only **300 epochs**.
- Train for **500 epochs**.
- Compare with the current **1000 epoch** model.
- Introduce a learning-rate scheduler after several hundred epochs.
- Evaluate downstream Variational Inference performance using each pretrained checkpoint.

Since the primary objective is to provide a good initialization for Variational Inference rather than minimize the supervised loss alone, shorter pretraining may provide similar downstream performance while reducing computation.

---

## Conclusions

### Positive Indicators

- Continuous loss reduction throughout training.
- Training and validation curves remain closely aligned.
- No evidence of overfitting.
- Stable optimization across all predicted fields.
- Surface elevation and thickness converge particularly well.
- Log-scale visualization confirms meaningful progress throughout optimization.

### Remaining Questions

- Is 1000 epochs necessary for effective VI initialization?
- Would earlier stopping (300–500 epochs) achieve comparable downstream performance?
- Would a learning-rate scheduler improve late-stage convergence?
- Is the higher noise in the **u** component due to scaling or intrinsic problem difficulty?

These experiments will help determine the most computationally efficient pretraining strategy before Variational Inference.

## Joint PINN + Variational Inference Training Analysis

<img width="1684" height="1516" alt="image" src="https://github.com/user-attachments/assets/0ec3d17f-708b-4c5b-b902-836bcfcdb193" />


The joint training stage appears stable, with both the training and validation ELBO remaining closely aligned throughout optimization. Unlike the PINN pretraining stage, the total loss changes only slightly because the pretrained network already provides a good initialization. Most of the optimization now focuses on balancing the data fit, PDE residual, and KL regularization rather than learning the solution from scratch.

---

### Key Observations

### 1. Total ELBO Loss

The total training and validation ELBO remain nearly identical throughout training.

Observations:

- Training and validation curves overlap almost perfectly.
- No systematic divergence between train and validation.
- The ELBO remains approximately constant throughout optimization.
- Occasional spikes occur in both curves simultaneously.

These spikes likely arise from stochastic optimization or minibatch sampling rather than instability, since both train and validation recover immediately afterward.

Overall, there is no evidence of overfitting or optimization failure.

---

### 2. Data Loss

The data loss remains centered around

$$
\log_{10}(\text{loss}) \approx -1.6
$$

with moderate stochastic fluctuations.

Observations:

- Small oscillations are expected during stochastic optimization.
- Several isolated spikes occur, but they are temporary.
- No long-term upward trend is observed.
- Training and validation data losses closely follow one another.

This indicates that the network maintains a consistent fit to the observed data throughout joint optimization.

---

### 3. PDE Residual

The PDE residual remains nearly constant throughout training.

Observations:

- Very little variation over time.
- Training and validation curves overlap.
- No indication that the physics constraint deteriorates.

This behavior suggests that the pretrained PINN already satisfies the governing equations reasonably well, so the optimization mainly preserves the learned physical consistency while refining the posterior.

---

### 4. Boundary Condition Loss

Boundary condition (BC) loss was not logged during this run because

```text
ssa_enforce_continuity = False
```

Therefore, only the data and PDE losses contribute to the reported optimization diagnostics.

---

## Interpretation

The behavior differs substantially from PINN pretraining.

During pretraining:

- Loss decreases rapidly as the network learns the glacier state variables.

During joint training:

- The pretrained PINN already provides a strong initialization.
- Optimization primarily adjusts the posterior distribution over viscosity.
- The objective balances:
  - Data loss
  - PDE residual
  - KL divergence (variational regularization)

Because the network begins near a good solution, only modest changes in the ELBO are expected.

---

## Positive Indicators

- Training and validation ELBO remain nearly identical.
- No evidence of overfitting.
- Stable optimization throughout the run.
- Data loss remains consistent.
- PDE residual stays approximately constant.
- Optimization quickly recovers after occasional stochastic spikes.

---

## Remaining Questions

- Does the ELBO truly converge, or has optimization plateaued?
- Would reducing the learning rate improve late-stage optimization?
- Is the KL divergence dominating the total loss?
- How much does joint training improve viscosity estimation relative to the pretrained PINN?
- Would logging the KL term separately clarify how the optimizer balances the ELBO components?

Further analysis of the individual ELBO components (data, PDE, and KL losses) and downstream viscosity prediction accuracy will help determine whether additional joint training provides meaningful improvements.

## log10(η) Prediction Error Analysis

<img width="947" height="372" alt="image" src="https://github.com/user-attachments/assets/6b1130a6-5ddc-49d8-ac1a-b1137114258d" />

The prediction errors for **log10(η)** indicate that the model exhibits a strong systematic bias while capturing little variation in the true viscosity field. The histogram shows a consistent underestimation of viscosity, and the scatter plot demonstrates almost no linear relationship between the predicted and reference values.

---

## Key Observations

### 1. Histogram of Prediction Errors

The histogram displays the distribution of

$$
\log_{10}(\eta_{\text{pred}}) - \log_{10}(\eta_{\text{ref}})
$$

Observations:

- The distribution is centered around **-1.0**.
- Nearly all errors are negative.
- Very few predictions have zero or positive error.
- The distribution is relatively narrow, indicating a consistent bias rather than random noise.

This suggests the model systematically predicts viscosities approximately **one order of magnitude lower** than the reference solution.

---

### 2. Scatter Plot

The scatter plot compares predicted and reference values.

Key observations:

- Correlation coefficient:
  - **r = -0.026**
- Points form a nearly horizontal band.
- No meaningful linear relationship exists between predictions and the reference values.
- Predictions remain clustered within a narrow range despite changes in the reference viscosity.

A correlation this close to zero indicates that the model is largely insensitive to the true spatial variability of viscosity.

---

## Interpretation

These results suggest that the model has learned a nearly constant or weakly varying estimate of viscosity rather than accurately recovering the underlying viscosity field.

Possible explanations include:

- The supervised signal for viscosity is too weak.
- The network is predicting the mean viscosity to minimize loss.
- Viscosity is not sufficiently identifiable from the available observations.
- Loss weighting favors other predicted variables over viscosity.
- Additional physics constraints may be required to distinguish viscosity variations.

---

## Potential Improvements

Possible next steps include:

- Increase the weighting of the viscosity-related loss term.
- Normalize viscosity targets to improve optimization.
- Examine spatial maps of predicted and reference viscosity.
- Evaluate viscosity error separately for grounded and floating ice.
- Compare viscosity performance before and after Variational Inference.
- Investigate whether PINN pretraining provides sufficient information for viscosity estimation.

---

## Conclusions

### Positive Indicators

- Prediction errors are relatively consistent across samples.
- No evidence of catastrophic numerical instability.
- Error distribution is well behaved without large outliers.

### Remaining Concerns

- Strong systematic underestimation of viscosity (~1 order of magnitude).
- Correlation between predictions and reference values is essentially zero.
- Model fails to capture spatial variability in viscosity.
- Indicates that additional training, improved loss formulation, or stronger physical constraints may be necessary for accurate viscosity recovery.
