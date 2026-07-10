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

## Plotting

<img width="1550" height="944" alt="image" src="https://github.com/user-attachments/assets/95de464b-e2b4-49ac-b080-2fb8c77bcc5b" />

The plot itself isn't evidence that training is bad—it mainly indicates that the visualization is masking most of the optimization because the initial loss is orders of magnitude larger than the final loss. However, it also raises the possibility that the PINN reached its useful minimum very early, which is exactly why the earlier note ("plot loss from VI to see if it saturated too soon") is important. A log-scale plot and separate loss components will tell me whether the model truly converged or whether the current plot is simply hiding meaningful progress.
