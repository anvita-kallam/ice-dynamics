## Objective: Resume VI Training After Wall-Time Limit

- Continue the Variational Inference training from the latest checkpoint after the Slurm job reached the cluster wall-time limit.

## Job Status Investigation

- Confirmed the training job did not crash; it was terminated by the scheduler after reaching the 12-hour wall-time limit (`ExitCode 0:10`, `SIGUSR1`).
- Verified that the latest checkpoint was successfully written before termination.
- Checked the saved checkpoint epoch and training logs to determine how many epochs remained.

## Training Configuration Updates

- Updated `run_torch.cfg` to resume training from the saved checkpoint:
  - `restore = True`
  - `restore_optimizer = True`
  - `n_epochs = 277` (remaining epochs after checkpoint at epoch 723)

## Slurm Configuration Updates

- Modified the training Slurm script to request a **24-hour** wall-time instead of 12 hours to reduce the number of restart cycles.
- Resubmitted the training job with checkpoint restoration enabled.

## Next Steps

- Monitor the resumed training to ensure it continues from epoch 723 rather than restarting.
- Once training completes, launch the prediction job to generate posterior inference results.
