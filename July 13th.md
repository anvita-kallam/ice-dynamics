# Model Hypertuning to Improve Joint PINN + Variational Inference Training

## Objective
The current joint PINN + Variational Inference implementation successfully minimizes the total ELBO, but it appears to be doing so by modifying the PINN state variables (velocity and thickness) instead of correctly inferring the latent viscosity field (η). The resulting viscosity predictions exhibit a strong systematic bias (approximately one order of magnitude too low) and essentially zero correlation with the reference viscosity field, despite stable optimization.

I want to modify the training procedure so that the optimizer is encouraged (or forced) to explain the observations through changes in viscosity rather than changes in velocity or thickness.

### Current Training Pipeline
