# Run Prediction + Start Plotting Model Training Performance

## Objective

- Run the prediction stage using the completed VI model to generate posterior viscosity estimates.
- Begin analyzing model performance by plotting training and validation losses over time.

## Prediction

- Submitted the prediction job using the trained VI checkpoint.
- Need to generate posterior predictions for the synthetic test case for comparison with the Icepack ground truth.


### Pipeline Status

| Stage | Status |
|--------|--------|
| Pretrain | ✅ Completed |
| VI Training | ✅ Completed (Job 978514) |
| Prediction | ⏳ Ready to submit |

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

