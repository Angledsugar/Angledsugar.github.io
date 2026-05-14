---
layout: page
title: Critical-Phase Detection for Vision-Language-Action Policies
description: Preprint, under review.
img: assets/img/projects/cpd/overview.png
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

We build a **zero-shot critical-phase detector** for frozen vision-language-action policies. Given just **5–20 expert demonstrations** and a frozen π₀ / π₀.₅ backbone, we automatically identify which rollout steps a robot is most likely to fail at — **no human labels, no task-specific reward, no per-task tuning**. On LIBERO-Long manipulation, we reach **leave-one-out F1 of 0.86–0.89** for trajectory-level failure detection.

<div class="row mt-4">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cpd/overview.png" title="CPD pipeline overview" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    <strong>Pipeline.</strong> A temporal-distance encoder φ maps raw proprioception to a 64-D task-progress latent. A self-supervised G2 labeler splits rollouts into success / failure buffers using only the geometry of the demo end-states. Per-buffer KDEs produce a Bayes-optimal log-ratio score r<sub>t</sub> at every step; a 3-step debounce marks the critical phase. Training (left, blue) is one-shot from demos; inference (right, red) is per-step on a fresh rollout.
</div>

## Why this matters

VLA models like π₀, OpenVLA, and RT-2 work _most of the time_ on common manipulation tasks — but they fail unpredictably on **precision phases**: the last few millimeters of insertion, the alignment before a grasp, the handoff between sub-tasks. Recent work (RLT, 2026) showed that **focusing RL fine-tuning on just these critical phases is dramatically more sample-efficient** than fine-tuning the whole trajectory.

The remaining question is _how to find them_. Existing answers all have a per-task cost:

| Approach                   | What it needs                            | What it costs                                  |
| -------------------------- | ---------------------------------------- | ---------------------------------------------- |
| Supervised labeling        | A human annotating each trajectory       | Time, doesn't scale                            |
| Ground-truth success label | A boolean "task done" predicate per task | Engineer-coded for each task; doesn't transfer |
| Reward-shaped RL           | A dense reward function                  | Task-specific, often impossible for VLA        |

**CPD removes all of these.** The detector reconfigures itself for a new task from **only the demos**, exploiting the fact that VLA backbones already encode "this state looks like task progress" — we just need to read it out.

## Method

### Temporal-distance latent representation (TLDR)

We learn a 64-D encoder φ on the proprioception of demos with a contrastive triplet loss: states that are close in **time within the same demo** are pulled together, states that are far apart are pushed apart. This collapses irrelevant variation (initial poses, recovery wiggles) and lines up trajectories on a 1-D progress coordinate.

<div class="row mt-3">
    <div class="col-sm-10 offset-sm-1">
        {% include figure.liquid loading="eager" path="assets/img/projects/cpd/triplet_sampling.png" title="Triplet sampling" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Triplet sampling within a demo: anchor s<sub>t<sub>a</sub></sub> is pulled toward a nearby positive s<sub>t<sub>p</sub></sub> (k<sub>pos</sub> = 2 steps ahead) and pushed away from a far negative s<sub>t<sub>n</sub></sub> (K<sub>neg</sub> ≥ 20 steps ahead). Both come from the same trajectory — no extra demos needed.
</div>

### G2 self-supervised labeler

Given a rollout from a frozen VLA policy, we don't know whether it succeeded — that's what an oracle predicate would tell us. Instead, we check whether the rollout's **final encoded state** lands inside the cluster of demo end-states in latent space:

$$
\ell_{G2}(\tau) = \mathbb{1}\big[\| \varphi(s_T) - g \| < \varepsilon \big]
$$

where g is the demo end-state centroid and ε is set automatically from demo statistics. **No tunable threshold, no per-task constants.** Theorem 1 in the paper proves this label converges to the ground-truth success predicate as the number of demos grows, under mild Lipschitz / ρ-cover assumptions on φ.

### Per-step critical score

Rollouts split by G2 into success buffer 𝐵<sub>+</sub> and failure buffer 𝐵<sub>−</sub>. We fit separate kernel density estimates and define

$$
r_t = \log \tilde{f}_+(z_t) - \log \tilde{f}_-(z_t)
$$

This is the Bayes-optimal log-likelihood ratio. A step is **critical** when r<sub>t</sub> < 0 for at least 3 consecutive steps (debouncing single-step noise). Trajectory-level rules — longest critical run, total critical-step count, critical fraction — are derived from this per-step signal.

## Results on LIBERO-Long (task 00, π₀.₅ backbone, 200 rollouts)

### Detection F1 scales with demos

<div class="row mt-3">
    <div class="col-sm-10 offset-sm-1">
        {% include figure.liquid loading="eager" path="assets/img/projects/cpd/loocv_f1_vs_n.png" title="LOO-CV F1 vs N" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Leave-one-out cross-validated F1 across three trajectory-level rules. F1 climbs from ~0.75 (N=10 demos) to ~0.88 (N=140) and then plateaus. The plateau is <strong>not</strong> a signal saturation — it's a denominator artifact: task 00 has only 13 failures in 200 rollouts, so the maximum recoverable F1 is bounded.
</div>

### Per-step critical phase: success vs failure cases

<div class="row mt-3">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/projects/cpd/cp_ep115_success.png" title="Clean success — ep115" class="img-fluid rounded z-depth-1" %}
        <div class="caption">
            <strong>ep115 (success).</strong> Only 7 critical steps (2.7%). CPD correctly says "fine".
        </div>
    </div>
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/projects/cpd/cp_ep068_failure.png" title="Clean failure — ep068" class="img-fluid rounded z-depth-1" %}
        <div class="caption">
            <strong>ep068 (failure).</strong> 93% critical steps, longest run = 485. The robot enters a critical region early and never escapes.
        </div>
    </div>
</div>

### Headline numbers

| Backbone | Task                | N demos | LOO-CV F1 | Reward separation (z-score) |
| -------- | ------------------- | ------- | --------- | --------------------------- |
| π₀.₅     | LIBERO-Long task 00 | 140     | **0.889** | 4.09                        |
| π₀.₅     | LIBERO-Long task 00 | 200     | 0.86      | 4.09                        |

The separation is essentially clean — the F1 ceiling is set by failure-pool size, not by signal quality.

## What's next

- Scaling to **harder tasks** with more failure modes (LIBERO-10 with weaker backbones) — needed to escape the small-failure-pool ceiling
- **Latent-space correction**: once a critical step is detected, project toward the nearest success-buffer latent and decode an action correction — converts CPD from a _detector_ to a _controller_
- Extension to **language-conditioned task transfer**: does the G2 ε threshold transfer across tasks within the same backbone?

<hr/>

## BibTeX {#bibtex}

```bibtex
@article{choi2026cpd,
  title   = {Critical-Phase Detection for Vision-Language-Action Policies},
  author  = {Choi, Chanyeok and Lee, Youngmoon},
  journal = {Preprint},
  year    = {2026},
  note    = {Under review},
}
```
