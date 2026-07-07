# Getting VI Code Running + Comparing SSA to Icepack

## Objective: Compare Archive VI Physics with the Icepack Forward Model

- Reviewed the Archive Variational Inference (VI) implementation and compared its governing equations to the Icepack SSA model used to generate the spin-up simulations.

## Physics Comparison

- Compared the momentum balance, rheology, basal friction law, continuity equation, solver formulation, and unit systems between the two implementations.
- Identified several key differences:
  - **Rheology:** Icepack uses nonlinear Glen's flow law, while the Archive VI model assumes a linear viscosity formulation.
  - **Basal friction:** Icepack spin-ups use a custom plastic hybrid friction law, whereas the Archive VI model uses a Coulomb-like sliding law with a learned sliding parameter.
  - **Units:** Icepack operates in MPa–m–yr units, while the Archive VI implementation assumes SI units (Pa–m–s).
  - **Continuity:** The Archive VI model enforces mass continuity as an additional residual, while Icepack only includes it during prognostic thickness evolution.

## Key Findings

- The current VI implementation does not solve the same physical equations used to generate the Icepack spin-up datasets.
- This mismatch likely introduces errors during inference, even when using the correct ground-truth viscosity fields.
- The largest sources of inconsistency are the unit system, basal friction formulation, and constitutive (rheology) model.

## Recommended Approach

- Keep the Icepack forward model unchanged since the production and test spin-up datasets have already been generated.
- Instead, modify the Archive VI implementation to better match the Icepack physics by:
  - Converting all quantities to Icepack's MPa–m–yr units.
  - Replacing the Archive basal friction law with the spin-up friction formulation.
  - Updating the membrane stress formulation to use Glen's flow law.
  - Removing the learned sliding parameter if the basal friction coefficient is fixed during spin-up.

## Code Updates

### `Archive/models_torch.py`
- Added helper functions for:
  - Icepack unit conversions.
  - Glen effective strain rate computation.
  - Glen effective viscosity.
  - Spin-up plastic basal drag law.
- Updated `_physics_nll_ssa` to:
  - Use Icepack units.
  - Compute membrane stresses using Glen rheology.
  - Remove the learned sliding parameter (`λ`) from the SSA residual.
  - Add a configuration option to either infer viscosity (`ssa_use_inferred_eta=True`) or compute it directly from Glen's law.

### `Archive/utilities_torch.py`
- Updated viscosity priors to MPa·yr units.
- Added Icepack SSA configuration parameters (`fluidity_A`, `friction_C`, Glen exponent, etc.).
- Preserved velocities in m/yr instead of converting to SI.
- Added automatic loading of the spin-up configuration (`A` and `C`) from the NPZ `cfg_json`.

### Additional Changes
- Updated `Archive/README.md` with documentation describing the SSA alignment.
- Added new configuration options for controlling continuity enforcement, rheology, and basal friction.

## Key Takeaways

- The Archive VI model now belongs to the same physics family as the Icepack forward model, reducing inconsistencies between the training data and inverse model.
- Basal sliding is now determined by the fixed spin-up friction coefficient rather than a learned sliding parameter.
- The implementation can now be run in either:
  - **Inverse mode**, where viscosity is inferred through VI, or
  - **Forward mode**, where viscosity is computed directly using Glen's law for validation.

## Next Steps

- Point the training configuration to the latest spin-up NPZ files so `A` and `C` are loaded automatically.
- Retrain the VI model using the updated physics and unit system.
- Compare inference results against the previous implementation to quantify the impact of the physics alignment.
