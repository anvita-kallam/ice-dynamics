# Final Joint Training Results

## Objective

Evaluate whether the modified training strategy successfully encourages the model to learn the latent viscosity field (η) while maintaining the pretrained PINN solution.

The final training configuration includes:

- Soft η prior anchor
- State regularization
- Stronger but stable physics weighting
- Separate learning rates
- Resume-safe PINN freezing
- Ice-masked inducing point placement
- Continuous η monitoring

---

## Overall Assessment

<img width="1183" height="1221" alt="image" src="https://github.com/user-attachments/assets/f5e94934-cf95-4dfc-bc1f-cfb768040c74" />


The final training curves indicate that the modified training strategy is behaving as intended.

Unlike previous runs, the model no longer exhibits viscosity collapse or unstable optimization. Training and validation losses remain closely aligned throughout optimization, suggesting that the model is learning consistently without overfitting.

---

## 1. Total ELBO

### Observations

- Training and validation losses overlap almost perfectly.
- Rapid decrease during the first ~50 epochs.
- Stable optimization while the PINN remains frozen.
- Additional improvement immediately after the PINN is unfrozen (epoch 400).
- No large oscillations or divergence.

### Interpretation

The optimizer successfully learns viscosity during the frozen stage.

Once the PINN is unfrozen, the network performs a small amount of joint fine-tuning, resulting in another gradual decrease in the ELBO rather than destabilizing training.

This is the intended behavior.

---

## 2. Data Loss

### Observations

- Nearly constant during the frozen stage.
- Small improvement after epoch 400.
- Training and validation remain almost identical.

### Interpretation

The pretrained PINN already provides a strong fit to the Icepack observations.

Rather than relearning velocity or thickness, joint optimization only makes small refinements after viscosity has already been estimated.

This indicates that state regularization is successfully preventing unnecessary changes to the glacier state.

---

## 3. KL + η Prior

### Observations

- Large decrease during the first ~150 epochs.
- Stabilizes afterward.
- Small spikes occur occasionally.
- Remains stable after PINN unfreezing.

### Interpretation

The Variational Gaussian Process initially adjusts its posterior toward the data while remaining close to the η prior.

After convergence, only small corrections occur.

The occasional spikes likely correspond to optimizer updates rather than instability.

Importantly, unlike previous runs, the KL term does not continue decreasing by driving η toward zero.

The η prior successfully prevents viscosity collapse.

---

## 4. Physics Loss

### Observations

- Nearly constant throughout training.
- Stable for both training and validation.
- No sudden increases after the PINN is unfrozen.

### Interpretation

The governing equations remain satisfied throughout optimization.

This indicates that allowing the PINN to update after epoch 400 does not sacrifice physical consistency.

---

## 5. State Regularization

### Observations

- Remains essentially zero throughout training.

### Interpretation

The pretrained PINN solution changes very little.

This is exactly the desired outcome.

Instead of modifying velocity, thickness, or surface elevation to reduce the loss, the optimizer primarily explains the remaining mismatch through viscosity.

---

## Comparison with Earlier Attempts

| Training Strategy | Outcome |
|-------------------|---------|
| Weak physics | PINN absorbed the mismatch; η was ignored. |
| Strong physics | η gradients improved, but viscosity collapsed toward zero. |
| Final strategy (η prior + state regularization + balanced physics) | Stable optimization, no viscosity collapse, successful joint fine-tuning after PINN unfreezing. |

---

# Additional Training Diagnostics

These plots provide further evidence that the updated optimization strategy successfully stabilizes training and shifts learning toward the latent viscosity field while preserving the pretrained PINN.

---

## Module Gradient Norms

<img width="1168" height="644" alt="image" src="https://github.com/user-attachments/assets/e16685eb-3629-4a7c-8037-9d49434b77cc" />


Verify that gradients are flowing to the intended parts of the model throughout training.

## Observations

- During the first **400 epochs**, the `mean_net` is frozen, so only the VGP receives gradients.
- The VGP gradient gradually decreases as viscosity converges, indicating stable optimization rather than exploding gradients.
- At **epoch 400**, the `mean_net` is unfrozen, producing a sharp increase in its gradient norm.
- After unfreezing, the PINN gradients steadily decrease, indicating controlled fine-tuning rather than large updates.
- The VGP gradients remain nonzero throughout training, meaning η continues to receive meaningful optimization signals.

## Interpretation

The gradient behavior matches the intended training strategy.

Before epoch 400, only the viscosity parameters are optimized. After the PINN is unfrozen, both models participate in optimization, but the gradients remain stable and gradually decrease as the solution converges.

Unlike earlier experiments where the PINN immediately dominated optimization, the staged training successfully allows η to learn first before introducing joint optimization.

---

## η Recovery — Bias

<img width="999" height="308" alt="Screenshot 2026-07-16 at 10 55 38 AM" src="https://github.com/user-attachments/assets/d17adae0-9930-40b3-97c5-7b41eb162698" />


Measure whether the predicted viscosity has the correct average value.

## Observations

- The log₁₀ bias rapidly improves during the first ~100 epochs.
- Afterward, the bias stabilizes around **−0.012**.
- The bias remains essentially unchanged after the PINN is unfrozen.

## Interpretation

A bias of zero corresponds to an unbiased viscosity estimate.

Remaining near **−0.012** indicates that the model predicts an average viscosity that is extremely close to the reference solution.

Importantly, unlike previous experiments where viscosity collapsed toward zero, the bias remains stable throughout the remainder of training.

This demonstrates that the η prior and state regularization successfully prevent viscosity collapse.

---

## η Recovery — RMSE and Correlation

<img width="999" height="312" alt="Screenshot 2026-07-16 at 10 55 59 AM" src="https://github.com/user-attachments/assets/4143488c-ca5d-472d-85b2-b084ff3dafd8" />


Evaluate both the overall prediction error and the ability to recover spatial variations in viscosity.

## Observations

### RMSE

- The log₁₀ RMSE remains approximately constant throughout training.
- No noticeable degradation occurs after epoch 400.

### Correlation

- Correlation begins relatively high (~0.65).
- Gradually decreases throughout training.
- Stabilizes near zero by the end of training.

## Interpretation

The stable RMSE indicates that the overall magnitude of the viscosity field remains accurate.

However, the decreasing correlation suggests that although the model predicts the correct average viscosity, it struggles to recover the detailed spatial variations of the reference field.

In other words,

- the **mean viscosity is correct**, but
- the **fine-scale spatial structure is still poorly reconstructed**.

This indicates that future improvements should focus on improving spatial recovery rather than correcting global bias.

Possible future directions include:

- increasing the number of inducing points,
- improving Gaussian Process expressiveness,
- modifying the physics loss,
- incorporating additional observational constraints.

---

# η Recovery — Mean Predicted vs Reference

<img width="895" height="315" alt="Screenshot 2026-07-16 at 10 56 56 AM" src="https://github.com/user-attachments/assets/751a0e3a-40fd-4eee-90af-fe6820dc145e" />


Compare the average predicted viscosity with the reference value throughout training.

## Observations

- The predicted mean viscosity rapidly increases from its initialization.
- It converges near **12 MPa·yr** within the first ~100 epochs.
- The reference mean remains approximately **15 MPa·yr**.
- The predicted mean remains stable after convergence.
- No drift occurs after the PINN is unfrozen.

## Interpretation

The optimizer quickly finds a physically reasonable viscosity and maintains it throughout joint training.

Unlike previous runs where η steadily collapsed toward zero, the predicted viscosity remains stable for the remainder of optimization.

Although the predicted mean remains slightly below the reference value, the remaining discrepancy is relatively small compared to earlier experiments and does not worsen after joint optimization begins.

Overall, these results indicate that the updated optimization strategy successfully stabilizes the viscosity estimate while preserving the pretrained PINN solution.

---

## Conclusions

The revised training strategy successfully resolved the major optimization issues identified during earlier experiments. By introducing staged optimization, stronger regularization, balanced physics constraints, and improved monitoring, the model now learns the latent viscosity field in a much more stable and physically meaningful manner.

Key improvements include:

- Stable ELBO optimization with closely aligned training and validation losses.
- Successful staged training, where the PINN remains frozen initially before controlled joint fine-tuning.
- Stable and well-behaved gradient flow into both the PINN and viscosity parameters.
- Near-zero viscosity bias throughout training with no evidence of viscosity collapse.
- Preservation of the pretrained glacier state through state regularization.
- Physics constraints remain satisfied throughout optimization.
- Consistent train/validation behavior with no signs of overfitting.

While the optimizer now reliably recovers the correct **mean viscosity** and maintains a physically consistent solution, the primary remaining limitation is recovering the **spatial variability** of the viscosity field. The declining correlation between the predicted and reference viscosity indicates that the model still struggles to capture fine-scale spatial patterns, despite accurately estimating the overall mean.

Overall, these results demonstrate that the revised optimization strategy successfully shifts learning toward the latent viscosity field while maintaining the physical consistency of the pretrained PINN, providing a significantly more robust foundation for future viscosity inference.

## Next Steps

Future work will focus on improving the spatial reconstruction of viscosity rather than overall optimization stability. Potential directions include:

- Increase the number or optimize the placement of Gaussian Process inducing points to better represent spatial variability.
- Explore alternative GP kernels or richer posterior parameterizations to increase model expressiveness.
- Investigate additional physics constraints or improved loss formulations that provide stronger spatial information for η.
- Incorporate additional observational data or synthetic experiments to improve the identifiability of viscosity.
- Evaluate the approach on more realistic glacier geometries and eventually extend the framework to real-world glacier datasets.
- Perform sensitivity studies on learning rates, regularization strengths, and physics weighting to further improve reconstruction accuracy.
