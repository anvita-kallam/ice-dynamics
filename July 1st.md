# Prepare VI datasets from spin-up NPZ

## Objective: Prepare Production Outputs for Variational Inference

- Create an interactive notebook for preprocessing production spin-up outputs into VI-ready datasets while keeping the preprocessing logic centralized in a reusable script.

### Notebook Development

- Created `notebooks/analysis/prep_vi_dataset.ipynb`, an interactive version of `scripts/prep_vi_dataset.py`.
- Structured the notebook to:
  - Import project paths and preprocessing functions.
  - Configure output directory, noise level, random seed, and production NPZ inputs.
  - Generate VI-ready datasets for both production cases.
  - Save `vi_case_*.npz` files and a `manifest.json` containing metadata and normalization statistics.

Manifest Summary
```
=== more_sliding (high sliding) ===
  source: outputs/spinup/production/more_sliding/SteadyState_more_sliding_10500yr_ramp4000_1refine_grid.npz
  bundle: outputs/vi/vi_case_more_sliding.npz
  C = 0.001 | grounded = 87.1%
  speed_obs: mean=193.4, std=237.2, max=1047
  viscosity_true: mean=16.02, std=9.318, max=42.55
  log_viscosity_true: mean=2.585, std=0.6468, max=3.751

=== no_sliding (low sliding) ===
  source: outputs/spinup/production/no_sliding/SteadyState_no_sliding_10500yr_ramp4000_1refine_grid.npz
  bundle: outputs/vi/vi_case_no_sliding.npz
  C = 100.0 | grounded = 88.9%
  speed_obs: mean=146.2, std=254.4, max=1064
  viscosity_true: mean=28.27, std=17.06, max=60.85
  log_viscosity_true: mean=3.068, std=0.8447, max=4.108

Normalization (pooled over grounded cells):
  speed_obs: {'mean': 169.55349572634748, 'std': 247.21565033452077}
  log_viscosity_true: {'mean': 2.8290880894361807, 'std': 0.7909941894994298}
  bed: {'min': -733.11634617052, 'max': 349.83232493477317}
```

### Updating VI Input Files

- Ran the preprocessing workflow using the latest production spin-up outputs.
- Updated `outputs/vi/manifest.json` to reference the new **10,500-year** production simulations instead of the previous **6,000-year** outputs.

### Key Outcome

- The preprocessing workflow now exists in both script and notebook form, allowing datasets to be regenerated interactively whenever new production spin-up outputs are available without duplicating preprocessing logic.

### Next Steps

- Use the generated VI datasets as inputs for the Variational Inference pipeline.
- Regenerate the VI dataset after future production spin-up runs.
