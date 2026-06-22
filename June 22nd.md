# Exploring the Codebase for Synthetic Ice Stream + Initial VI Model

## Objective
Understand Dr.Emma Liu's given code and test under constrained circumstances

## spinupNewFull (9 steps)
- Build coarse mesh with MeshPy/Triangle (~500 m resolution)
- Spin up CG1 on coarse mesh for 6500 yr (coarse_total_time)
- Spin up CG2 on the same coarse mesh (independent run, not a continuation of CG1)
- Estimate error as |h_CG2 − h_CG1|
- Smooth error into a DG0 field (L2 projection with facet penalty)
- Refine mesh — shrink element areas where error is high (typically near the grounding line)
- Spin up CG1 on the fine mesh, initialized from the coarse CG2 solution
- Spin up CG2 on the fine mesh, also from coarse CG2
- Save outputs via save_checkpoint_and_grid_npz:
  * {output_stem}.h5 — mesh + fields (velocity, thickness, surface, bed, A)
  * {output_stem}.json — full config
  * {output_stem}_grid.npz — fields on a regular 500 m grid

