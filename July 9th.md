# Resume VI Training + Evaluate Equation Loss Changes

## Objective: Resume VI Training After Wall-Time Limit

- Continue the Variational Inference training from the latest checkpoint after the Slurm job reached the cluster wall-time limit.

### Job Status Investigation

- Confirmed the training job did not crash; it was terminated by the scheduler after reaching the 12-hour wall-time limit (`ExitCode 0:10`, `SIGUSR1`).
- Verified that the latest checkpoint was successfully written before termination.
- Checked the saved checkpoint epoch and training logs to determine how many epochs remained.

### Training Configuration Updates

- Updated `run_torch.cfg` to resume training from the saved checkpoint:
  - `restore = True`
  - `restore_optimizer = True`
  - `n_epochs = 277` (remaining epochs after checkpoint at epoch 723)

### Slurm Configuration Updates

- Modified the training Slurm script to request a **24-hour** wall-time instead of 12 hours to reduce the number of restart cycles.
- Resubmitted the training job with checkpoint restoration enabled.

### Next Steps

- Monitor the resumed training to ensure it continues from epoch 723 rather than restarting.
- Once training completes, launch the prediction job to generate posterior inference results.

## Discussion Notes / Next Steps

### Variational Inference Training
- Plot the VI training loss to determine whether optimization is saturating too early.
- Increase VI training duration to **5k–10k epochs** (rather than ~100 epochs).
- Freeze the pretrained PINN during the initial VI epochs, then jointly optimize the PINN and VI model.

### Loss Function Improvements
- Current objective:
  - Equation loss
  - Data loss
  - KL divergence
- Data loss should directly compare PINN predictions (velocity/thickness) against Icepack outputs.
- Updated equation loss now incorporates the Icepack-aligned physics.

### Model Behavior
- The optimizer can reduce the total loss by modifying **velocity/thickness** instead of **viscosity**, since it is not explicitly encouraged to separate these contributions.
- Preliminary attempts at reweighting the loss terms were inconclusive and require further testing.
- Explore configuration parameters and training strategies that constrain the model to primarily learn **η (viscosity)** while keeping the PINN solution closer to the forward model.

### Validation Strategy
- Benchmark the current VI implementation on a synthetic case with known viscosity.
- Even if reconstruction error is relatively high, proceed to experiments on real glacier data afterward.
- Drop the **no-sliding** case for now and focus on the more-sliding configuration.

### Miscellaneous
- AGU Conference (San Francisco): abstract deadline **August 5**.
- WAIS application will focus on **glaciers only**.
