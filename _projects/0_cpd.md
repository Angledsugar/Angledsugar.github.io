---
layout: page
title: Self-Supervised Critical Phase Detection for VLA Refinement
description: Preprint, under review.
img: assets/img/projects/cpd/hero.png
importance: 0
category: research
abbr: 2026
year: 2026
authors: Chanyeok Choi, Youngmoon Lee
github: https://github.com/Angledsugar/critical-phase-detector
related_publications: true
---

<div class="row">
  <div class="col-12 text-center">
    <p class="mb-1"><a href="/">Chanyeok Choi</a><sup>1*</sup>, <a href="https://sites.google.com/umich.edu/youngmoonlee/home">Youngmoon Lee</a><sup>1</sup></p>
    <p class="text-muted mb-3" style="font-size: 0.9em;"><sup>1</sup> Hanyang University</p>
    <p class="mb-3">
      <span class="badge bg-primary" style="font-size: 0.95em;">Preprint · Under review</span>
    </p>
    <p>
      <a class="btn btn-outline-primary btn-sm mx-1" href="#bibtex" role="button">📄 BibTeX</a>
      <a class="btn btn-outline-primary btn-sm mx-1" href="https://github.com/Angledsugar/critical-phase-detector" role="button" target="_blank">💻 Code</a>
      <a class="btn btn-outline-secondary btn-sm mx-1" href="#" role="button" aria-disabled="true">📑 Paper (coming soon)</a>
    </p>
  </div>
</div>

<hr/>

## TL;DR

A frozen vision-language-action (VLA) policy can _believe_ it has finished a sub-task while the end-effector is still kinematically stalled — peg hovering above the hole, gripper closed on air. We call this failure mode **premature commitment** and detect it _step-by-step_ from demos alone, with **no labels, no success oracle, and no per-task reward**.

The detector reads two estimators of the same quantity — normalized task progress — off two different sensors. The VLA's action-expert pre-head hidden state gives us the policy's **belief** about progress; proprioception gives us the **physical** progress. Their disagreement $\delta_t = \hat\pi_t - \pi^*_t$ is the **execution gap**, and $\delta_t > 0$ marks a step where the latent has run ahead of the kinematics. On top of the same signal we add a $\sim 1\text{K}$-parameter residual adapter that refines actions only inside the detected critical phases.

<div class="row mt-4">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cpd/hero.png" title="Execution-gap pipeline" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    <strong>Pipeline.</strong> Proprioceptive state s<sub>t</sub> and the VLA action-expert pre-head hidden z<sup>vla</sup><sub>t</sub> are independently mapped onto a shared demo phase space [0, 1] by two small self-supervised estimators f<sub>s</sub>, f<sub>z</sub>. The execution gap δ<sub>t</sub> = π̂<sub>t</sub> − π*<sub>t</sub> is a per-step scalar; δ<sub>t</sub> > τ marks a critical phase, and a low-rank residual adapter refines actions only there.
</div>

## Demos

Per-step critical-phase detection on two LIBERO-Long rollouts, π₀.₅ backbone. Top: rollout frames synced to the timeline. Middle / bottom: smoothed progress rate and gap magnitude. Shaded bands are the detected critical phases (where $\delta_t$ exceeds the demo-derived 95-quantile threshold).

<div class="row mt-3">
    <div class="col-md-6">
        {% include video.liquid path="assets/video/cpd_bottleneck_demo000.mp4" class="img-fluid rounded z-depth-1" controls=true autoplay=true loop=true muted=true preload="metadata" poster="/assets/img/projects/cpd/bottleneck_demo000.png" %}
        <div class="caption">
            <strong>Rollout #000.</strong> Critical phases concentrate around the contact-rich grasp and the final alignment.
        </div>
    </div>
    <div class="col-md-6">
        {% include video.liquid path="assets/video/cpd_bottleneck_demo250.mp4" class="img-fluid rounded z-depth-1" controls=true autoplay=true loop=true muted=true preload="metadata" poster="/assets/img/projects/cpd/bottleneck_demo250.png" %}
        <div class="caption">
            <strong>Rollout #250 (T = 388).</strong> Multiple critical phases — the policy keeps committing prematurely whenever a new sub-task begins.
        </div>
    </div>
</div>

## Why precision phases break VLAs

VLA backbones like π₀, π₀.₅, OpenVLA, and RT-2 handle the bulk of a manipulation rollout — then fail unpredictably on the last few millimeters. Looking step-by-step, two visually similar but semantically different dwellings appear:

| Phase                      | What the policy is doing                                          | What the kinematics show       | Should we intervene?                         |
| -------------------------- | ----------------------------------------------------------------- | ------------------------------ | -------------------------------------------- |
| **Compliant fine-control** | "I'm in a contact-rich phase, slow down." Latent advances slowly. | EE moves slowly, in agreement. | **No.** This is correct behavior.            |
| **Premature commitment**   | "Sub-goal done, move on." Latent advances.                        | EE is stalled or misaligned.   | **Yes.** Policy is acting on a false belief. |

Region-binary success metrics (LIBERO's "did the gripper end inside the goal region?") compress this distinction away — a 0.5 mm misalignment that downstream recovers from still counts as success, and the step where the policy first committed prematurely is invisible. Step-level detection is the prerequisite for step-level refinement.

## Self-supervised gap detection

The central observation is asymmetric: **kinematic state cannot lie, VLA latent can**. Joint encoders report the actual end-effector pose up to sub-mm noise. The VLA's internal latent is a _belief_ — under distribution shift (sensor noise, contact uncertainty, novel object pose) it can be wrong, and "I have finished aligning" is exactly the kind of wrong it can be.

We read both signals as estimators of the _same_ quantity — normalized demo phase $p \in [0, 1]$ — so they are directly comparable. Training is self-supervised on demo time itself:

$$
\mathcal{L}_z = \sum_{n, t} \bigl(f_z(z^{\text{vla},(n)}_t) - t/T_n\bigr)^2, \qquad
\mathcal{L}_s = \sum_{n, t} \bigl(f_s(s^{(n)}_t) - t/T_n\bigr)^2.
$$

No external labels, no success oracle. The critical phase set uses a quantile threshold over the demo distribution:

$$
\mathcal{C} = \{t : \delta_t > \tau\}, \quad \tau = \mathrm{Quantile}_{0.95}\bigl(\{\delta_t^{\text{demo}}\}\bigr).
$$

<div class="row mt-3">
    <div class="col-sm-10 offset-sm-1">
        {% include figure.liquid loading="eager" path="assets/img/projects/cpd/compare_defs.png" title="Comparing critical-phase definitions" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Comparing critical-phase definitions on a single LIBERO-Long rollout. The bottom panel attributes each step to BN-only (kinematic-only signal), KDE-only (latent-only signal), Both, or neither — motivating why the <em>disagreement</em> between the two sensors is the right step-level quantity rather than either sensor alone.
</div>

## Gap-to-failure chain (Theorem 1)

The detector is justified by a short logical chain. Under three natural assumptions:

- **(A1)** Kinematic state is the ground truth of physical reality (up to measurement noise).
- **(A2)** Demo trajectories are samples of the success manifold $\mathcal{M}^*$.
- **(A3)** VLA action is a function of internal latent: $a_t \sim p(a \mid z^{\text{vla}}_t)$.

**Theorem 1 (gap-to-failure consistency).** _Under (A1)–(A3), if $\delta_t > 0$ over interval $[t_1, t_2]$ with positive cumulative gap, then $s_{t*2} \notin \mathcal{M}^\*$.*

The proof is an 8-step chain in which each implication is justified by exactly one of the three assumptions. Intuition: the VLA outputs the action that belongs to phase 0.85 (per A3); that action only completes its intended transition when the kinematic state is also at phase 0.85 (per A2); but the kinematics are actually at 0.70 (per A1) — so the transition fails and the state leaves the demo manifold. When (A2) is weakened to "demo phase is one sufficient condition among many" — free-space pick, multi-modal goals — the chain generalizes to a probabilistic **Theorem 1′**.

## Residual refinement with a hybrid gate

Detection alone is not the goal. On top of the frozen VLA we add a rank-8 residual adapter ($\approx 1\text{K}$ trainable parameters, zero-initialized):

$$
a_t = \pi_{\text{base}}(o_t) + g_t \cdot \Delta a_t, \quad
\Delta a_t = W_{\text{up}} \, \mathrm{ReLU}(W_{\text{down}} \, \phi(o_t)).
$$

The design question is _when_ the adapter releases control back to the base policy. Two desiderata are in tension:

- **(P) Pareto-safety.** Outside $\mathcal{C}$, $a_t = \pi_{\text{base}}(o_t)$ **bit-exact** — not "small," but identically the base action.
- **(S) Boundary smoothness.** Around the entry/exit of $\mathcal{C}$, small observation perturbations must produce small action changes — $a$ has a finite Lipschitz constant.

We prove that **neither mechanism alone suffices**:

| Mechanism                                                                              | Property it guarantees                           | Why the other can't replace it                                                                        |
| -------------------------------------------------------------------------------------- | ------------------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| Sparsity penalty $\lambda \, \|\Delta a_t\|^2$ in training loss                        | $\|\Delta a\| \le G / (2\lambda)$ — gives (S)    | Lemma 1: ReLU MLPs are generically nonzero on every open set, so no penalty can force bit-exact zero. |
| Hysteresis hard gate $g_t \in \{0, 1\}$, band $\tau_{\text{low}} < \tau_{\text{high}}$ | Bit-exact zero outside $\mathcal{C}$ — gives (P) | Lemma 2: at a regular boundary point, a hard switch with unbounded magnitude has Lipschitz $\infty$.  |

**Lemma 3** then gives the explicit hybrid bound:

$$
L \le L_{\text{base}} + L_\Delta + \frac{G \, \|\nabla \delta\|_\infty}{4 \, \lambda \, \epsilon},
$$

where $\epsilon = (\tau_{\text{high}} - \tau_{\text{low}}) / 2$ is the hysteresis half-band. Both knobs enter the same bound — the trade-off between correction strength and smoothness becomes explicit and tunable.

The intuition box in the appendix maps this onto lane-keeping assist: magnitude limiting without an on/off switch leaks corrective torque even during clean driving (Lemma 1); on/off without magnitude limiting jerks the wheel when it engages (Lemma 2); production LKA combines both (Lemma 3).

## Experiments (in progress)

|           | Value                                                                |
| --------- | -------------------------------------------------------------------- |
| Benchmark | LIBERO-Long (`libero_10`), contact-rich subset                       |
| Backbones | π₀, π₀.₅_libero (frozen), action-expert pre-head hook                |
| Demos     | $N = 50$ per task ($500$ across 10 tasks)                            |
| Adapter   | rank $r = 8$, $\approx 1\text{K}$ trainable params, backbone frozen  |
| Training  | model-based dynamics rollout, or PPO with $r_t = -\max(0, \delta_t)$ |

Primary metrics:

- **(P) verification** — $\|a_t - \pi_{\text{base}}(o_t)\|_\infty < 10^{-7}$ on every $g_t = 0$ step.
- **(S) verification** — measured Lipschitz constant inside the Lemma 3 bound.
- **Refinement effectiveness** — $\delta_t$ reduction inside $\mathcal{C}$; success-rate delta vs no-adapter baseline; EE alignment error in mm.
- **Phase discrimination** — z-score / ROC-AUC separating compliant fine-control from premature commitment.

## What's next

- Probabilistic Theorem 1′ on free-space / multi-modal goal tasks.
- Cross-task phase-predictor generalization on LIBERO-10 with weaker backbones.
- Real-robot transfer (LIBERO sim → physical robot), where (A1) sensor lie-impossibility needs quantitative validation under partial observability.
- Task-adaptive $(\tau_{\text{high}}, \tau_{\text{low}}, k, \lambda)$.

<hr/>

## BibTeX {#bibtex}

```bibtex
@article{choi2026cpd,
  title   = {Self-Supervised Critical Phase Detection for VLA Refinement},
  author  = {Choi, Chanyeok and Lee, Youngmoon},
  journal = {Preprint},
  year    = {2026},
  note    = {Under review},
}
```
