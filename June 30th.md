## Further Analysis of NPZ output (thickness over time, SI units)

### Objective
- Verify that the glacier reaches a steady state after the basal sliding coefficient (`C`) is ramped by tracking how key quantities evolve over time.

### Work Completed
- Began plotting the evolution of **ice velocity** and **mean ice thickness** throughout the simulation.
- Focused on identifying whether these quantities stabilize after the ramp period.

### Key Idea
- A simulation is considered to have reached **steady state** when quantities such as velocity and thickness stop changing significantly with time.
- If the velocity continues changing after the ramp, the glacier has not yet fully equilibrated and may require a longer simulation.

### Next Steps
- Analyze the velocity and thickness time series for both sliding cases.
- Confirm that both simulations have reached equilibrium before using their outputs for downstream analyses and Variational Inference.
