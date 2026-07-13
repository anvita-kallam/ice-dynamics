# Model Hypertuning to Improve Joint PINN + Variational Inference Training

## Objective
The current joint PINN + Variational Inference implementation successfully minimizes the total ELBO, but it appears to be doing so by modifying the PINN state variables (velocity and thickness) instead of correctly inferring the latent viscosity field (η). The resulting viscosity predictions exhibit a strong systematic bias (approximately one order of magnitude too low) and essentially zero correlation with the reference viscosity field, despite stable optimization.

I want to modify the training procedure so that the optimizer is encouraged (or forced) to explain the observations through changes in viscosity rather than changes in velocity or thickness.

### Current Training Pipeline

Current workflow:

1. Pretrain PINN on Icepack observations (velocity, thickness, etc.).
2. Jointly optimize the pretrained PINN together with the Variational Inference model.
3. Total loss is

$$
L = L_{\text{data}} + L_{\text{PDE}} + L_{\text{KL}}
$$

where

- **Data loss:** difference between PINN predictions and Icepack outputs.
- **PDE loss:** momentum / governing equation residual.
- **KL loss:** variational regularization.

Currently, the optimizer is free to reduce the loss by changing either

- viscosity (desired), or
- velocity/thickness fields (undesired).

The optimization seems to choose the easier path of modifying the glacier state variables rather than recovering the correct viscosity.
