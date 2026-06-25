## Analyze the Production SpinUps and Understand the Physics

## Objectives
- Analyze the outputs from the two completed Icepack spin-up simulations with extreme basal sliding coefficients.
- Compare steady-state thickness, speed, effective viscosity, and grounding line location.

### Work Completed
- Reviewed the comparison notebook and interpreted the generated plots.
  ```
  def plot_pair(field_key, title, explanation, cmap="viridis", diff_cmap="RdBu_r"):
    """Plot the same physical field for both sliding cases and their difference."""
    a = more[field_key]  # more sliding field
    b = less[field_key]  # less sliding field
    X, Y = more["X"], more["Y"]
    diff = a - b  # positive → more sliding case is larger there

    more_c = more["cfg"]["C"]
    less_c = less["cfg"]["C"]

    fig, axes = plt.subplots(1, 3, figsize=(16, 4.2))
    fig.suptitle(explanation, fontsize=11, y=1.12)

    side_panels = [
        (a, f"More sliding\nC = {more_c:g}\n{title}"),
        (b, f"No sliding\nC = {less_c:g}\n{title}"),
    ]

    # Left two panels: absolute fields on the MISMIP+ 640 km × 80 km domain.
    for ax, (field, subtitle) in zip(axes[:2], side_panels):
        im = ax.pcolormesh(X / 1e3, Y / 1e3, field, shading="auto", cmap=cmap)
        ax.set_title(subtitle, fontsize=10)
        ax.set_xlabel("Downstream x (km)")  # x = 0 at inflow, x = 640 km at terminus
        ax.set_ylabel("Across-flow y (km)")
        ax.set_aspect("equal")
        cbar = fig.colorbar(im, ax=ax, fraction=0.046, pad=0.02)
        cbar.ax.set_ylabel(title.split("(")[-1].rstrip(")"), rotation=270, labelpad=12)

    # Right panel: case separation map used to judge ground-truth distinguishability.
    vmax = np.nanmax(np.abs(diff))
    if vmax == 0:
        vmax = 1.0
    im = axes[2].pcolormesh(
        X / 1e3,
        Y / 1e3,
        diff,
        shading="auto",
        cmap=diff_cmap,  # red = more sliding larger, blue = no sliding larger
        vmin=-vmax,
        vmax=vmax,
    )
    axes[2].set_title("Difference\n(more sliding − no sliding)", fontsize=10)
    axes[2].set_xlabel("Downstream x (km)")
    axes[2].set_aspect("equal")
    cbar = fig.colorbar(im, ax=axes[2], fraction=0.046, pad=0.02)
    cbar.ax.set_ylabel("Δ " + title.split("(")[0].strip(), rotation=270, labelpad=12)

    # Quick numeric read on whether the two cases are meaningfully different.
    mean_diff = float(np.nanmean(diff))
    rms_diff = float(np.sqrt(np.nanmean(diff**2)))
    max_abs = float(np.nanmax(np.abs(diff)))
    axes[2].text(
        0.02,
        0.98,
        f"mean Δ = {mean_diff:+.3g}\nRMS Δ = {rms_diff:.3g}\nmax |Δ| = {max_abs:.3g}",
        transform=axes[2].transAxes,
        va="top",
        ha="left",
        fontsize=9,
        bbox=dict(boxstyle="round", facecolor="white", alpha=0.85),
    )

    plt.tight_layout()
    plt.show()
  ```
- Compared the final states for the high-sliding (`C = 0.0001`) and no-sliding (`C = 100`) simulations.
- Found that the two simulations produced nearly identical steady-state thickness, speed, viscosity, and grounding line positions despite the large difference in basal sliding.

### Key Takeaways
- The final equilibrium glacier geometry appears largely insensitive to these extreme basal sliding values.
- Most differences are small and localized near the grounding line.
- Prepared questions about whether transient evolution may contain a stronger signal than the final steady-state fields for viscosity inference.

### Next Steps
- Discuss these results with my mentor during our 1:1.
- Determine whether to analyze transient outputs or additional diagnostic fields.
