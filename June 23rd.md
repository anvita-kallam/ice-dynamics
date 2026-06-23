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

### Current Status
Both notebooks have been patched with the updated convergence strategy and output configuration. The next step is to rerun the simulations and evaluate whether the gradual ramping approach successfully reaches the desired sliding extremes while maintaining solver convergence.
