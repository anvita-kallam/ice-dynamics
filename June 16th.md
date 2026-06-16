# Firedrake & Icepack Environment Setup

## Objective
Set up and verify a working Firedrake and Icepack computational environment for glacier modeling research.

## Setup Completed
- Installed **Firedrake** (main branch) and **Icepack 1.1.0** using the `firedrake-conda` repository.
- Created a Conda environment at `~/firedrake-env` with all required dependencies, including PETSc, MPI, and HDF5.
- Verified the installation using `verify.py`; all **31 tests passed**, including:
  - Firedrake imports
  - JIT compilation
  - Poisson equation solve
  - Icepack model functionality

## Issues Encountered

### Path Names with Spaces
The initial installation failed because the environment was created inside a directory containing a space (`Ice Dynamics`). This caused compiler wrappers such as `mpicc` to fail.

**Resolution:** Recreated the environment in a path without spaces:

```bash
~/firedrake-env
```

## Version Notes
- The default Firedrake release (`2025.10.2`) was incompatible with Conda PETSc `3.25.2` due to an API mismatch involving `DMPlexFilter`.
- Installed **Firedrake main + firedrake-rtree** instead, which is compatible with the current PETSc version and works reliably on Apple Silicon systems.

## Verification Results
All 31 checks in `verify.py` completed successfully, including:
- Firedrake imports
- Icepack imports
- JIT compilation
- Finite-element solves
- Icepack flow models

## Usage

### Activate Environment

```bash
export PATH="$HOME/miniforge3/bin:$PATH"
eval "$(conda shell.bash hook)"
conda activate ~/firedrake-env
```

### Verify Installation

```bash
python "/Users/anvitakallam/Ice Dynamics/firedrake-conda/verify.py"
```

### Test Imports

```bash
python -c "import firedrake; import icepack; print('Ready')"
```

## Environment Configuration
The following variables are automatically configured when activating the environment:

- `PETSC_DIR`
- `CC=mpicc`
- `HDF5_MPI`
- Other MPI/PETSc-related settings

## Source Clones
Local source repositories are stored in:

```bash
~/.firedrake-conda/clones/
```

To update Firedrake and Icepack:

```bash
conda activate ~/firedrake-env

pip install --upgrade --no-build-isolation \
  -c $CONDA_PREFIX/constraints.txt \
  ~/.firedrake-conda/clones/firedrake \
  ~/.firedrake-conda/clones/icepack
```

## Outcome
A fully functional Firedrake/Icepack environment was successfully installed and validated. The environment is ready for glacier flow simulations, PDE-based modeling, and ice rheology parameter inference research.
