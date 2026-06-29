## Analyze the Re-Ran Spinups post 10500 year update

### Objective
- Analyze results now that total duration is higher to ensure ground truth matches physical inferences

## Production Spin-up Results (10,500 yr, 4,000 yr Ramp)

### Simulation Settings

| Parameter | More Sliding | No Sliding |
|-----------|--------------|------------|
| Basal sliding coefficient (`C`) | 0.001 | 100.0 |
| Starting `C` | 0.01 | 0.01 |
| Ramp time | 4,000 yr | 4,000 yr |
| Total stage time | 10,500 yr | 10,500 yr |
| Time step | 0.25 yr | 0.25 yr |

### Results

| Metric | More Sliding | No Sliding |
|--------|--------------:|-----------:|
| Mean thickness | 448.28 m | 1046.58 m |
| Maximum speed | 999.32 m/yr | 1011.55 m/yr |
| Mean viscosity | 14.9 | 25.9 |
| Grounded area | 87.1% | 88.9% |
| NaNs | 0 | 0 |

### Differences (More Sliding − No Sliding)

- **Mean thickness:** -598.30 m (RMS = 736.04 m)
- **Mean speed:** +44.39 m/yr (maximum absolute difference = 617.30 m/yr)
- **Mean viscosity:** -11.0 (RMS = 16.95)

<img width="1069" height="700" alt="Screenshot 2026-06-29 at 9 21 05 AM" src="https://github.com/user-attachments/assets/92674c37-ebbd-4f80-95ef-20c95c7cf262" />


### Key Findings

- After updating the production settings (10,500-year simulation with a 4,000-year ramp), the two simulations diverged much more than in the previous comparison.
- The **more-sliding** case produced a substantially **thinner glacier**, **lower effective viscosity**, and a **slightly smaller grounded area** than the **no-sliding** case.
- While the maximum ice speeds remained similar, the spatial speed differences became much larger, indicating that basal sliding had a stronger influence on glacier flow.
  - The mean speed difference is positive, meaning the glacier is, on average, flowing faster in the more-sliding case.
- These results show that the updated spin-up configuration preserved the expected physical differences between the two basal sliding regimes, providing a much clearer signal for downstream analysis.
