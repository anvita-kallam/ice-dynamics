# How I tuned the viscosity (η) model

This note is about recovering ice **viscosity** η on the synthetic “more sliding”
MISMIP+ spin-up, using variational inference (VI) with a frozen PINN.

---

## 1. Goal

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

<img width="2737" height="1182" alt="image" src="https://github.com/user-attachments/assets/ab5fc445-8f1f-43e6-9372-dbbdaef4876a" />


**Takeaway:** sequential recovery follows the true spatial pattern much more
closely (correlation **r ≈ 0.83** vs **r ≈ 0.68** for joint).


The optimized sequential baseline still had a clear problem: **η was too low
on average** (mean ≈ 8–9 vs truth ≈ 14.9). Spatial pattern was good; magnitude
was biased low.

<img width="2041" height="1059" alt="image" src="https://github.com/user-attachments/assets/167589cb-f053-4384-b748-41fa5995baad" />


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

<img width="2482" height="1243" alt="image" src="https://github.com/user-attachments/assets/3824d65e-003c-4dc5-9452-e23cb783d789" />


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

<img width="2041" height="1059" alt="image" src="https://github.com/user-attachments/assets/0d6ab87f-812d-4399-abe0-e563882fe09c" />



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

<img width="2041" height="1059" alt="image" src="https://github.com/user-attachments/assets/2783e86e-62c0-4184-9ed4-a9e77a9f05ca" />

---

## 5. Uncertainty: are the error bars trustworthy?

For η_init=15 we plotted where the truth falls inside the model’s ±1σ band.

<img width="2105" height="587" alt="image" src="https://github.com/user-attachments/assets/d1450b3f-edca-4485-a28c-2b23fb822bdb" />


- About **83%** of points are inside 1σ (a well-calibrated Gaussian would be ~68%).
- That means this model is a bit **conservative**: error bars are a little wide,
  which is usually safer than being overconfident.
- Misses (red) cluster in the central high-η band — the same place the
  estimate is still a bit too smooth / too low.

Scatter of estimate vs truth for the same run:

<img width="1892" height="557" alt="image" src="https://github.com/user-attachments/assets/ebf418f5-57f8-4e93-9b65-19a30bbabc25" />

