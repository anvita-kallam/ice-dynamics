# How we tuned the viscosity (η) model

This note walks through the **model-tuning story** in plain language.
It is about recovering ice **viscosity** η on the synthetic “more sliding”
MISMIP+ spin-up, using variational inference (VI) with a frozen PINN.

Figures live under [`outputs/figures/vi/`](../../outputs/figures/vi/).
Technical suite details: [`ETA_BIAS_SUITE.md`](ETA_BIAS_SUITE.md).
Pipeline overview: [`SEQUENTIAL_VI_PIPELINE.md`](SEQUENTIAL_VI_PIPELINE.md).

---

## 1. What we are trying to do

Ice flow depends on how “stiff” the ice is. That stiffness is the viscosity η.
We do **not** observe η directly. We observe velocity and geometry, then ask
a statistical model to infer η.

On this test problem we **do** know the true η from the spin-up, so we can
score how good the recovery is:

| Metric | Meaning |
|--------|---------|
| **Correlation r** | Do high/low η regions line up spatially? (higher is better) |
| **log₁₀ bias** | Is the estimate systematically too low or too high? (closer to 0 is better) |
| **Mean η** | Average viscosity; truth is about **14.9** MPa·yr |
| **1σ coverage** | How often does the truth fall inside the model’s ±1σ band? (~68% would be ideal for a perfect Gaussian) |

---

## 2. Step A — Train the flow model first, then learn η

We use a **sequential** recipe:

1. **Pretrain** a neural network (PINN) to match the ice state (velocity, surface, thickness).
2. **Freeze** that network.
3. Run **VI only on η** (a Gaussian process over log-viscosity).

This beat the older approach of training the PINN and the viscosity model
**together** (“joint” training).

![Joint vs sequential η maps](../../outputs/figures/vi/joint_vs_sequential_optimized/eta_joint_vs_sequential_maps.png)

**Takeaway:** sequential recovery follows the true spatial pattern much more
closely (correlation **r ≈ 0.83** vs **r ≈ 0.68** for joint).

![Joint vs sequential diagnostics](../../outputs/figures/vi/joint_vs_sequential_optimized/eta_joint_vs_sequential_diagnostics.png)

The optimized sequential baseline still had a clear problem: **η was too low
on average** (mean ≈ 8–9 vs truth ≈ 14.9). Spatial pattern was good; magnitude
was biased low.

![Optimized sequential η vs truth](../../outputs/figures/vi/more_sliding_vi_only_optimized/eta_truth_estimate_diff.png)

---

## 3. Step B — An eta-bias suite: change one knob at a time

We built eight isolated experiments from the optimized sequential config.
Each run keeps its own checkpoints so nothing overwrites the baseline.

| Experiment | What we changed (in one sentence) |
|------------|-----------------------------------|
| `control_adamw` | Repeat the optimized recipe (sanity check) |
| `weak_prior` | Trust the data more; loosen the η prior |
| `raised_prior_center` | Start the prior mean at **15** instead of **12** |
| `strong_physics` | Weight SSA physics residuals more heavily |
| `gp_capacity` | Bigger / richer Gaussian-process kernel |
| `optimizer_ngd` | Different optimizer for the variational GP |
| `optimizer_fast_cosine` | Higher learning rate, shorter schedule |
| `combined_candidate` | Several of the above at once |

Eligibility gate for “good enough spatial skill”: **r ≥ 0.82**.

### Side-by-side: truth vs four finished suite models

![Truth vs four suite estimates](../../outputs/figures/vi/eta_bias_v1_truth_vs_four_estimates.png)

**What the eye sees:** every estimate is still darker (lower η) than truth, but
**raised center (η_init=15)** is the least dark of this set. Weak prior is the
most under-estimated.

### What we learned from those knobs

- **Raising the prior center** helped the most: mean η went up and correlation
  stayed high.
- **Weakening the prior** made the low-η bias *worse* — the physics residual
  was already pushing η down, so loosening the anchor let it fall further.
- **Stronger physics** also pushed η down.
- Optimizer / GP-capacity variants did not beat a simple raised prior on this
  problem (their finished bests stayed below η_init=15 / 18).

---

## 4. Step C — Raise the prior again (η_init = 15 → 18)

Because 12 → 15 helped, we tried **η_init = 18**.

![η_init=18 truth vs estimate](../../outputs/figures/vi/raised_prior_center_18/eta_truth_estimate_diff.png)

Full-grid scorecard (best checkpoint):

| Model | r | log₁₀ bias | Mean η | 1σ coverage |
|-------|---|------------|--------|-------------|
| Control (η_init=12) | 0.831 | −0.176 | 8.3 | ~0.78 |
| Raised (η_init=15) | 0.836 | −0.101 | 9.9 | **~0.83** |
| Raised (η_init=18) | **0.845** | **−0.032** | **11.6** | ~0.50 |

**On this synthetic test:** η_init=18 is the best match to truth
(highest correlation, smallest bias, highest mean).

**Caveat:** its uncertainty got overconfident (1σ coverage fell to ~50%).
η_init=15 is a better “honest uncertainty” model.

![η_init=15 recovery](../../outputs/figures/vi/raised_prior_center_15/eta_truth_estimate_diff.png)

---

## 5. Uncertainty: are the error bars trustworthy?

For η_init=15 we plotted where the truth falls inside the model’s ±1σ band.

![η_init=15 1σ coverage](../../outputs/figures/vi/raised_prior_center_15/eta_1sigma_coverage.png)

- About **83%** of points are inside 1σ (a well-calibrated Gaussian would be ~68%).
- That means this model is a bit **conservative**: error bars are a little wide,
  which is usually safer than being overconfident.
- Misses (red) cluster in the central high-η band — the same place the
  estimate is still a bit too smooth / too low.

Scatter of estimate vs truth for the same run:

![η_init=15 uncertainty / scatter](../../outputs/figures/vi/raised_prior_center_15/eta_uncertainty_scatter.png)
