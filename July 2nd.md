# VI Modelling Test + Icepack SSA Discovery

## Objective: Develop a Variational Inference Training Pipeline

- Build an end-to-end workflow for inferring glacier viscosity from noisy surface velocity observations using Variational Inference (VI).

### VI Training Notebook Development

- Created `notebooks/learning/train_vi_viscosity_test.ipynb`.
- Implemented a workflow to:
  - Load test spin-up outputs from `TEST_CASES`.
  - Generate VI input bundles (`speed_obs`, masks, and ground-truth fields).
  - Train a field-based VI model to infer log-viscosity from noisy velocity observations.
  - Save posterior results to `outputs/vi/test/results_<case_id>.npz`.
  - Visualize posterior mean vs. ground-truth viscosity in both log and linear space.
  - Summarize inference results for both test cases.

### Variational Inference Model Implementation

- Created `scripts/vi_viscosity_model.py`.
- Implemented a prototype VI pipeline that:
  - Fits a local surrogate relating log-speed to log-viscosity using the forward-model snapshot.
  - Performs factorized Gaussian Variational Inference on the grounded ice domain using only noisy velocity observations.
  - Interpolates the inferred posterior back to the full glacier domain.
  - Returns posterior mean, posterior standard deviation, and evaluation metrics (R² and RMSE).

### Initial Model Performance

| Case | R² (log η) | RMSE (log η) |
|------|-----------:|-------------:|
| More Sliding | ~0.26 | ~0.64 |
| No Sliding | ~0.41 | ~0.82 |

<img width="1459" height="377" alt="Screenshot 2026-07-02 at 9 42 18 AM" src="https://github.com/user-attachments/assets/0026e538-ec73-43e0-afec-65a76d9f7c2b" />
<img width="1467" height="386" alt="Screenshot 2026-07-02 at 9 42 39 AM" src="https://github.com/user-attachments/assets/01b436f8-05b9-4c6d-93c1-d4a28c8af524" />

### Problems

- ❌ The posterior is much smoother than the ground truth.
- ❌ It underestimates the high-viscosity region near the left.
- ❌ Fine-scale spatial variability is almost entirely missing.
- ❌ The residual maps show systematic bias rather than random error. Large blue regions indicate the model consistently predicts viscosities that are too low in that area.

This suggests the model is learning the average field rather than the detailed spatial variations.

More Sliding vs. No Sliding
- Interestingly:
  - No Sliding has the higher R² (~0.41), meaning it captures spatial variation somewhat better.
  - But it also has a worse RMSE (~0.82), indicating larger absolute prediction errors.

### Key Takeaways

- Successfully established a complete prototype VI workflow from preprocessing through posterior inference and visualization.
- The current implementation is an emulator-based proof of concept using NumPy and SciPy, where the surrogate model replaces the forward physics during inference.
- A future physics-informed implementation will replace the surrogate likelihood with Icepack's `diagnostic_solve` to perform inference directly using the glacier dynamics model.

## Exact SSA Equations of Icepack

Since the VI needs to use the exact same equations as Icepack, these are necessary to uncover
- read the source code at https://github.com/icepack/icepack

## Strain rate and Glen constitutive law

### Strain-rate tensor

In plan-view (xy) models, icepack defines the horizontal strain-rate tensor as the symmetric gradient:

\[
\dot\varepsilon = \frac{1}{2}\left(\nabla u + \nabla u^{\mathsf T}\right)
\]

(`icepack.calculus.sym_grad`)

### Effective strain rate (regularized)

To avoid singularities at zero strain rate:

\[
\dot\varepsilon_e = \sqrt{\frac{1}{2}\left(\dot\varepsilon : \dot\varepsilon + (\mathrm{tr}\,\dot\varepsilon)^2 + \dot\varepsilon_{\min}^2\right)}
\]

### Glen flow law

\[
\dot\varepsilon = A\,|\tau|^{n-1}\,\tau
\]

where **τ** is the deviatoric stress tensor and **A** is the temperature-dependent fluidity (scalar field in icepack).

### Membrane stress tensor

For depth-averaged SSA, icepack uses a **membrane stress** tensor **M**:

\[
M = 2\mu\left(\dot\varepsilon + (\mathrm{tr}\,\dot\varepsilon)\,I\right),
\qquad
\mu = \tfrac{1}{2}\,A^{-1/n}\,\dot\varepsilon_e^{\,1/n - 1}
\]

(`icepack.models.viscosity.membrane_stress`)

The depth-averaged Cauchy stress is related to **M** through the vertical integration that produces the SSA system.

---

## Action functional (general form)

All icepack diagnostic models minimize:

\[
J = J_{\text{visc}} + J_{\text{fric}} + J_{\text{side}} - J_{\text{grav}} - J_{\text{term}} + J_{\text{penalty}}
\]

(terms absent in a given model are omitted)

| Term | Physical meaning |
|------|------------------|
| J_visc | Viscous dissipation (membrane / Glen) |
| J_fric | Basal sliding friction (grounded only) |
| J_side | Sidewall drag along fjord walls |
| J_grav | Gravitational driving (surface slope or buoyancy) |
| J_term | Stress work at calving / grounding line |
| J_penalty | Penalty for normal flow at sidewalls |

The velocity **u** satisfies **δJ/δu = 0** (weak form solved by Newton in `FlowSolver.diagnostic_solve`).

---

## Viscous term (shared by `IceStream` and `IceShelf`)

From `viscosity_depth_averaged()`:

\[
J_{\text{visc}} = \int_\Omega \frac{2n}{n+1}\, h\, A^{-1/n}\, \dot\varepsilon_e^{\,1/n + 1}\, \mathrm{d}x
\]

The GMD paper (Eq. 10) writes an equivalent form:

\[
J_{\text{visc}} = \int_\Omega \frac{n}{n+1}\, h\, A^{-1/n}\, |\dot\varepsilon|^{1/n + 1}\, \mathrm{d}x
\]

The two expressions use slightly different tensor-norm conventions but represent the same Glen dissipation.

With **n = 3**:

\[
J_{\text{visc}} = \int_\Omega \frac{3}{2}\, h\, A^{-1/3}\, \dot\varepsilon_e^{\,4/3}\, \mathrm{d}x
\]

---

## Grounded SSA — `IceStream`

### Total action

\[
J = J_{\text{visc}} + J_{\text{fric}} + J_{\text{side}} - J_{\text{grav}} - J_{\text{term}} + J_{\text{penalty}}
\]

### Basal friction (Weertman law)

Basal shear stress:

\[
\tau_b = -C\,|u|^{1/m - 1}\,u
\]

Friction contribution to the action (`bed_friction`):

\[
J_{\text{fric}} = -\frac{m}{m+1} \int_\Omega \tau_b \cdot u\,\mathrm{d}x
= \frac{m}{m+1} \int_\Omega C\,|u|^{1/m + 1}\,\mathrm{d}x
\]

With **m = 3**: basal drag magnitude scales as **C |u|²**.

Smaller **C** → more sliding. The Ice Dynamics spin-up cases use small **C** (more sliding) vs large **C** (effectively no sliding).

### Gravitational driving

\[
J_{\text{grav}} = -\int_\Omega \rho_I\, g\, h\, \nabla s \cdot u\,\mathrm{d}x
\]

This is the work done by the driving stress **ρ_I g h ∇s** against the ice flow.

### Calving / marine terminus (grounded)

\[
J_{\text{term}} = \int_\Gamma \left(\frac{1}{2}\rho_I g h^2 - \frac{1}{2}\rho_W g d^2\right) u \cdot \nu\,\mathrm{d}\gamma
\]

where:

- **Γ** is the ice front boundary
- **ν** is the outward unit normal
- **d = min(s − h, 0)** is water depth (sea level at z = 0)

### Sidewall friction (optional)

On boundary IDs marked as sidewalls:

\[
J_{\text{side}} = -\frac{m}{m+1} \int_\Gamma h\,\tau_s(u_t) \cdot u_t\,\mathrm{d}\gamma
\]

where **u_t** is the velocity component tangent to the wall and **τ_s** has the same Weertman form as basal friction with coefficient **C_s**.

### Strong-form momentum balance (grounded)

Taking the first variation of **J** with respect to **u** gives the standard MacAyeal SSA system:

\[
\nabla \cdot (h\sigma) + \rho_I g h\,\nabla s - \tau_b = 0
\]

where **σ** is the depth-averaged membrane stress derived from Glen's law, and **τ_b = C|u|^{1/m−1} u**.

In words: **divergence of membrane stress** + **driving stress** = **basal drag**.

---

## Floating SSA — `IceShelf`

For floating ice:

- Basal friction **C = 0**
- Hydrostatic surface: **s = (1 − ρ_I/ρ_W) h**
- icepack uses an integrated-by-parts form where the shelf gravity and terminus terms are related

Define the **buoyancy-adjusted density**:

\[
\varrho = \rho_I\left(1 - \frac{\rho_I}{\rho_W}\right)
\]

### Buoyancy / gravity

\[
J_{\text{grav}} = -\frac{1}{2}\int_\Omega \varrho\, g\, \nabla(h^2) \cdot u\,\mathrm{d}x
\]

### Terminus

\[
J_{\text{term}} = \frac{1}{2}\int_\Gamma \varrho\, g\, h^2\, u \cdot \nu\,\mathrm{d}\gamma
\]

### Strong-form momentum balance (floating)

\[
\nabla \cdot (h\sigma) + \varrho\, g\, h\,\nabla h = 0
\]

This is the shelf SSA balance: membrane stress divergence balances the buoyancy-driven driving stress.

---

## Prognostic thickness equation

SSA diagnostic solves are coupled to thickness evolution via the continuity equation (`icepack.models.transport.Continuity`):

\[
\frac{\partial h}{\partial t} + \nabla \cdot (h u) = \dot a_s - \dot a_b
\]

| Symbol | Meaning |
|--------|---------|
| ẋ_a_s | Surface mass balance |
| ẋ_a_b | Basal mass balance |

icepack's prognostic solver truncates **h** to zero where it becomes negative (approximate ice margin tracking).

---

## Boundary conditions

| Type | Implementation |
|------|----------------|
| **Inflow Dirichlet** | Prescribe **u** where ice flows in |
| **Calving / terminus** | Natural (Neumann) via **J_term** |
| **Sidewalls** | No normal flow via penalty; tangential drag via **J_side** |
| **Outflow** | Natural where ice flows out |

---
