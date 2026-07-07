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

## Next Steps

- Begin aligning the Archive VI code with the Icepack forward model, starting with unit conversions and the basal friction formulation before updating the rheology.
