## Further Analysis of NPZ output (thickness over time, SI units)

### Objective
- Verify that the glacier reaches a steady state after the basal sliding coefficient (`C`) is ramped by tracking how key quantities evolve over time.

### Work Completed
- Began plotting the evolution of **ice velocity** and **mean ice thickness** throughout the simulation.
- Focused on identifying whether these quantities stabilize after the ramp period.

<img width="955" height="640" alt="Screenshot 2026-06-30 at 9 21 07 AM" src="https://github.com/user-attachments/assets/a7810cc1-bd63-424d-af66-695a702d16f6" />

  
```
from parse_spinup_log import (
    find_latest_run_log,
    parse_spinup_log,
    stage_local_years,
    stage_points,
    series_or_none,
)


def _stage_ramp_end_yr(stage_pts, cfg):
    """Stage-local time when C reaches target, if that happens within this stage."""
    if not stage_pts:
        return None
    c_target = float(cfg["C"])
    t0 = stage_pts[0].t_yr
    t_end = stage_pts[-1].t_yr - t0
    for point in stage_pts:
        if point.C >= 0.99 * c_target:
            local = point.t_yr - t0
            if local < 0.99 * t_end:
                return local
            return None
    return None


def plot_spinup_steady_state(case_label, case, log_points, *, color):
    stage = "fine_high"
    stage_pts = stage_points(log_points, stage)
    if len(stage_pts) < 3:
        print(f"{case_label}: not enough {stage} diagnostics in log")
        return None

    t_local, h_vals = stage_local_years(stage_pts)
    speeds = series_or_none(stage_pts, "avg_speed")
    ramp_end = _stage_ramp_end_yr(stage_pts, case["cfg"])

    fig, axes = plt.subplots(1, 2, figsize=(12, 3.8), sharex=True)
    axes[0].plot(t_local, h_vals, color=color, lw=1.5)
    axes[0].set_ylabel("Mean thickness (m)")
    axes[0].set_title("Thickness", fontsize=10)

    if speeds is not None:
        axes[1].plot(t_local, speeds, color=color, lw=1.5)
        axes[1].set_ylabel("Mean speed (m/yr)")
        axes[1].set_title("Speed", fontsize=10)
    else:
        dh_dt = np.gradient(h_vals, t_local)
        axes[1].plot(t_local, np.abs(dh_dt), color=color, lw=1.5)
        axes[1].set_ylabel("|dh/dt| (m/yr)")
        axes[1].set_title("Thickness tendency (speed not in log)", fontsize=10)

    if ramp_end is not None:
        for ax in axes:
            ax.axvline(ramp_end, color="0.45", ls="--", lw=1.0)
        axes[0].text(
            ramp_end,
            axes[0].get_ylim()[1],
            "  C ramp end",
            va="top",
            ha="left",
            fontsize=8,
            color="0.35",
        )

    for ax in axes:
        ax.set_xlabel("Stage time (yr)")
        ax.grid(True, alpha=0.25)

    ramp_yr = case["cfg"].get("C_ramp_time")
    stage_yr = case["cfg"].get("coarse_total_time")
    fig.suptitle(
        f"{case_label}: {stage} stage  |  C ramp = {ramp_yr:g} yr, stage = {stage_yr:g} yr",
        fontsize=11,
        y=1.05,
    )
    plt.tight_layout()
    return fig


for case_label, case, color in (
    ("More sliding", more, "tab:blue"),
    ("No sliding", less, "tab:red"),
):
    log_path = find_latest_run_log(ROOT, case["cfg"]["case_id"])
    if log_path is None:
        print(f"{case_label}: no run.log found in outputs/logs/spinup/")
        continue
    print(f"{case_label}: {log_path.relative_to(ROOT)}")
    log_points = parse_spinup_log(log_path)
    fig = plot_spinup_steady_state(case_label, case, log_points, color=color)
    if fig is not None:
        plt.show()
```
### Velocity Plot
- The existing production logs do not include avg speed=, so the plot uses a |dh/dt| proxy (dashed line, noted in the panel title)

<img width="1148" height="617" alt="Screenshot 2026-06-30 at 9 33 25 AM" src="https://github.com/user-attachments/assets/f9d84501-1e99-4374-8ee7-c1825e8260da" />

### Key Idea
- A simulation is considered to have reached **steady state** when quantities such as velocity and thickness stop changing significantly with time.
- If the velocity continues changing after the ramp, the glacier has not yet fully equilibrated and may require a longer simulation.

### Next Steps
- Analyze the velocity and thickness time series for both sliding cases.
- Confirm that both simulations have reached equilibrium before using their outputs for downstream analyses and Variational Inference.
