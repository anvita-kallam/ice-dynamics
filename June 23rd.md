# Getting Basal Sliding Bounds Closer to 0 and 1

## Objective
Mitigate convergence issue by testing both decreased time step and increased time step

## Progress Update: Sliding Extremes Convergence Testing

### Overview
Updated both spin-up notebooks to improve solver stability and convergence when testing extreme basal sliding conditions. The notebooks now use a gradual parameter ramping strategy and additional solver safeguards to handle highly stiff configurations.

### Sliding Regimes Tested

Because the IceStream model uses a regularized sliding law, it cannot represent true no-slip (0% sliding) or free-slip (100% sliding) conditions. Instead, numerical extremes were used:

| Case | Target Sliding Parameter (C) | Interpretation |
|--------|--------|--------|
| moreSlide | 1e-4 | Very weak basal resistance (maximum sliding) |
| lessSlide | 1e2 | Strong basal resistance (near-frozen bed) |

Both simulations begin from a stable initial value of **C = 1e-2** to avoid introducing an excessively stiff problem at the start of the run.

### Modifications Implemented

#### 1. Gradual Sliding Parameter Ramping
- Implemented logarithmic ramping of the basal sliding parameter.
- C transitions from **1e-2** to the target value over approximately **3000–4000 years** per stage.
- This reduces solver instability caused by abrupt parameter changes.

#### 2. Solver Fallback Strategy
- Added bidirectional fallback behavior for diagnostic solves.
- If the primary solver fails, the notebook automatically attempts the alternative solver (PETSc or Icepack).

#### 3. Reduced Timestep
- Decreased timestep size to **dt = 0.25 years**.
- Improves numerical stability at the cost of approximately 4× longer runtimes.

#### 4. Relaxed Newton Solver Settings
- Increased maximum Newton iterations to **500**.
- Set convergence tolerance to **1e-6**.
- Provides additional flexibility for difficult nonlinear solves.

#### 5. Separate Output Directories
- Configured unique output locations for each sliding experiment to prevent file overwrites and result collisions.

| Notebook | Output Prefix | Output Directory |
|-----------|--------------|------------------|
| `spinupNewFull-moreSlide.ipynb` | `SteadyState_more_sliding_*` | `adaptivity_output_more_sliding/` |
| `spinupNewFull-lessSlide.ipynb` | `SteadyState_no_sliding_*` | `adaptivity_output_no_sliding/` |

## Test Mode Implementation

Added a configurable switch:

```python
TEST_MODE = True
```

to enable rapid validation of the pipeline before launching full production runs.

| Setting | Test Mode | Production Mode |
|----------|------------|----------------|
| Stage length | 50 yr | 6500 yr |
| Purpose | Workflow verification | Quasi-steady-state solution |
| Runtime | ~15–30 min | Several hours |
| Outputs | `*_test_50yr_*` | `*_6000yr_*` |

Based on discussion and preliminary results, the production duration may potentially be reduced to approximately 4000 years if thickness evolution has stabilized after the friction ramp completes.

---

## Previous Production Run

Prior to splitting the workflow into separate extreme-sliding cases, a full production simulation was completed using the default friction value (`C = 1e-2`).

Generated outputs:

```text
SteadyState_MeshRefinement_6000yr_1refine.h5
SteadyState_MeshRefinement_6000yr_1refine.json
SteadyState_MeshRefinement_6000yr_1refine_grid.npz
```

This run exercised the full adaptive mesh-refinement workflow and produced physically reasonable glacier states.

---

## Extreme-Sliding Test Results

Both extreme-sliding test cases completed successfully today.

### More Sliding Case

Output:
`SteadyState_more_sliding_test_50yr_1refine_grid.npz`

| Metric | Value |
|----------|----------|
| C | 1e-4 |
| NaNs | 0 |
| Mean Thickness | 128.18 m |
| Mean Speed | 11.95 m/yr |
| Mean Viscosity | 258 |
| Grounded Fraction | 61.5% |

### Less Sliding Case

Output:
`SteadyState_no_sliding_test_50yr_1refine_grid.npz`

| Metric | Value |
|----------|----------|
| C | 1e2 |
| NaNs | 0 |
| Mean Thickness | 128.20 m |
| Mean Speed | 11.34 m/yr |
| Mean Viscosity | 340 |
| Grounded Fraction | 60.9% |

### Validation Outcome

Pipeline verification passed successfully.

Both simulations completed:

- Four-stage spin-up sequence
- Adaptive mesh refinement
- Diagnostic solves
- Checkpoint generation
- Grid export

No convergence failures or NaN values were observed.

---

## Interpretation of Results

The test runs demonstrate that the convergence modifications successfully allow simulations to reach extreme sliding configurations without solver failure.

Observed physical trends are consistent with expectations:

- Higher sliding produces greater ice velocity.
- Higher sliding corresponds to lower inferred viscosity.
- Grounding-line and thickness fields remain physically realistic.

However, differences between the two cases remain relatively small:

- Thickness RMS difference ≈ 2.4 m
- Mean velocity difference ≈ 0.6 m/yr

This is expected because the 50-year test only serves as a pipeline verification run. Most of the simulation time is spent ramping the friction parameter, leaving little time for the glacier to equilibrate at the target sliding value.

Therefore, the current test outputs should not be considered final synthetic ground truths.

---

## Outstanding Work

The following tasks remain:

### Milestone 1
- Run full production spin-ups (`TEST_MODE = False`) for both sliding scenarios.
- Generate final production `.npz` ground-truth datasets.
- Compare outputs using `analyze_spinup_npz.ipynb`.

### Milestone 2
- Develop viscosity inference / variational inference workflow.

### Milestone 3
- Apply methodology to real glacier datasets.

### Milestone 4
- Prepare poster and final project presentation materials.

---

## Next Steps

1. Set `TEST_MODE = False` in both spin-up notebooks.
2. Run both production simulations overnight.
3. Analyze production outputs using `analyze_spinup_npz.ipynb`.
4. Use the resulting production `_grid.npz` files as synthetic ground-truth datasets for viscosity inference experiments.
5. Defer use of `runSimNew.ipynb` unless future forward perturbation experiments are required.

