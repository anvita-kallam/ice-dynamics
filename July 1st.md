# Prepare VI datasets from spin-up NPZ

### Objective: Prepare Production Outputs for Variational Inference

- Create an interactive notebook for preprocessing production spin-up outputs into VI-ready datasets while keeping the preprocessing logic centralized in a reusable script.

### Notebook Development

- Created `notebooks/analysis/prep_vi_dataset.ipynb`, an interactive version of `scripts/prep_vi_dataset.py`.
- Structured the notebook to:
  - Import project paths and preprocessing functions.
  - Configure output directory, noise level, random seed, and production NPZ inputs.
  - Generate VI-ready datasets for both production cases.
  - Save `vi_case_*.npz` files and a `manifest.json` containing metadata and normalization statistics.

### Updating VI Input Files

- Ran the preprocessing workflow using the latest production spin-up outputs.
- Updated `outputs/vi/manifest.json` to reference the new **10,500-year** production simulations instead of the previous **6,000-year** outputs.

### Key Outcome

- The preprocessing workflow now exists in both script and notebook form, allowing datasets to be regenerated interactively whenever new production spin-up outputs are available without duplicating preprocessing logic.

### Next Steps

- Use the generated VI datasets as inputs for the Variational Inference pipeline.
- Regenerate the VI dataset after future production spin-up runs.
