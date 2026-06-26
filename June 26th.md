## Alter Production Sninup to see Increased Differences

## Objectives
- Try to reduce ramping and see if that produces viable output

## Spin-up Timing Updates

### Original Production Settings (Both Cases)

| Parameter | Value |
|-----------|-------|
| `coarse_total_time` / `fine_total_time` | 10,500 yr |
| `C_ramp_time` | 4,000 yr |
| `coarse_dt` / `fine_dt` | 0.25 yr |
| Time at target `C` per stage | 6,500 yr (62%) |

### Test Settings (Both Cases)

| Parameter | Value |
|-----------|-------|
| `TEST_STAGE_YEARS` | 200 yr |
| Scaled `C_ramp_time` | ~76.2 yr |
| Post-ramp equilibration | ~123.8 yr |

### Notes

- Tested reducing the ramp time to shorten the transition to the target basal sliding coefficient.
- The shorter ramp produced stable results for the **more-sliding** case but not for the **no-sliding** case.
- To keep both simulations consistent and allow sufficient equilibration, I instead increased the total simulation time for both cases so they aligned under the same production settings.
