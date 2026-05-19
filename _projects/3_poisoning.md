---
layout: page
title: Poisoning Attacks on Multi-Agent Reinforcement Learning Systems
description: Humanoids 2025 (Late-Breaking Report).
img: assets/img/projects/poisoning/poisonkiller_setup.png
importance: 3
category: research
abbr: 2025
year: 2025
authors: Chanyeok Choi, Jaehwan Cho, Youngmoon Lee
related_publications: true
---

<div class="row">
  <div class="col-12 text-center">
    <p class="mb-1">
      <a href="/">Chanyeok Choi</a><sup>1*</sup>, Jaehwan Cho<sup>1</sup>, <a href="https://sites.google.com/umich.edu/youngmoonlee/home">Youngmoon Lee</a><sup>1</sup>
    </p>
    <p class="text-muted mb-3" style="font-size: 0.9em;"><sup>1</sup> Hanyang University</p>
    <p class="mb-3">
      <span class="badge bg-primary" style="font-size: 0.95em;">Humanoids 2025 · Late-Breaking Report</span>
    </p>
    <p>
      <a class="btn btn-outline-primary btn-sm mx-1" href="#bibtex" role="button">📄 BibTeX</a>
      <a class="btn btn-outline-primary btn-sm mx-1" href="https://github.com/RAISELab/MAPA" role="button" target="_blank">💻 Code</a>
    </p>
  </div>
</div>

<hr/>

## TL;DR

A **reward-poisoning attacker agent**, trained jointly inside a multi-agent RL system, can scatter high-reward _lure points_ that drag a converged cooperative policy off-trajectory — without ever touching weights or other agents' observations. On a Unity 50×50 m crawler/lure benchmark, the same attack drops cumulative reward by **18.7% (PPO)** and **20.9% (SAC)** in the multi-agent setting; in the single-agent setting SAC collapses entirely (from 1276 → 23.93). The asymmetry has a structural cause: PPO's on-policy clipping locks the policy onto whichever lure it samples first, while SAC's off-policy + maximum-entropy replay dilutes poison samples — except when the buffer is too small to outvote a persistent attacker.

<div class="row mt-4">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/poisoning/poisonkiller_setup.png" title="Crawler/attacker environment" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    <strong>Setup.</strong> 50×50 m Unity environment. <em>Left</em>: baseline — crawler agents (blue stars) navigate toward green target points to maximize cumulative reward. <em>Right</em>: under attack — attacker agents place red lure points at arbitrary locations; a crawler that touches a lure receives a deceptively high reward, redirecting its policy away from the true target.
</div>

## Threat model

Two agent classes coexist in one environment with **predefined, fixed reward rules**:

| Agent        | Goal                                                | Reward structure                                           |
| ------------ | --------------------------------------------------- | ---------------------------------------------------------- |
| **Crawler**  | Reach the green target, maximize cumulative reward. | +100 on touching a lure (the trap); −1 on removing a lure. |
| **Attacker** | Maximize crawler distraction.                       | +1 each time a crawler is lured.                           |

The attacker has no access to crawler weights, no privileged sensors, no offline corruption of training data. It interacts only through the environment, by placing lure points — the same channel any other agent uses. This is what makes the attack realistic: any adversary that can _participate_ in a shared MARL environment can poison it.

The attacker's playbook has three components:

1. **Reward manipulation** — inject premature rewards before the true goal, subtly altering reward _timing_ so monitoring systems see plausible learning curves.
2. **Random behavior** — relocate the goal object to unreachable positions to break determinism and starve the crawler of stable signal.
3. **Tempting-reward (lure) attack** — sprinkle artificially high-reward points along plausible paths, exploiting reward-addiction dynamics to collapse exploration onto the trap.

## Why PPO and SAC respond differently

The paper's headline result is that **PPO is structurally more vulnerable to lure-style poisoning in multi-agent settings**, despite SAC showing a marginally larger absolute drop in the multi-agent column. The reason is mechanical:

- **PPO** is on-policy with a clipped surrogate objective. Once a lure perturbs the rollout distribution, the next batch over-samples the lure region; the clip then constrains policy updates _around the current (lure-biased) policy_. The result is **self-reinforcing trap capture** — the policy can't take a large enough step to escape the basin.
- **SAC** is off-policy with maximum-entropy regularization. The replay buffer dilutes poisoned samples across thousands of clean transitions, and the entropy term forces continued exploration around any apparent optimum. Lure capture requires the attacker to flood the buffer faster than it cycles.

The single-agent SAC collapse (1276 → 23.93) is the exception that proves the rule: with one crawler and one attacker, the buffer fills slowly enough that even a small number of lure samples become the dominant signal.

## Results

Cumulative reward at 1M training steps, averaged across crawler / attacker agents:

| Scenario         | Agent    | Baseline |       Under attack |
| ---------------- | -------- | -------: | -----------------: |
| Multi-Agent PPO  | Crawler  |    528.4 | **429.4** (−18.7%) |
| Multi-Agent PPO  | Attacker |        — |             −2.903 |
| Multi-Agent SAC  | Crawler  |    971.3 | **769.9** (−20.9%) |
| Multi-Agent SAC  | Attacker |        — |             −2.449 |
| Single-Agent PPO | Crawler  |    647.5 | **302.5** (−53.3%) |
| Single-Agent PPO | Attacker |        — |                 +1 |
| Single-Agent SAC | Crawler  |     1276 | **23.93** (−98.1%) |
| Single-Agent SAC | Attacker |        — |             −31.43 |

<div class="row mt-3">
    <div class="col-sm-10 offset-sm-1">
        {% include figure.liquid loading="eager" path="assets/img/projects/poisoning/poisoning_result.png" title="Average reward under baseline vs. attack" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    <strong>Reward curves.</strong> Average reward versus training steps for PPO and SAC in single-agent (top) and multi-agent (bottom) environments, with and without the attacker. Attack runs show larger variance and lower asymptotes; the gap is most pronounced for single-agent SAC, where the buffer is too small to wash out lure samples.
</div>

## What this means for deployed MARL

Cooperative MARL is at the core of human-interactive robotics — humanoid teams, robot taxis, drone fleets. All of them share the same exposed surface: a reward signal that comes from the environment, not from a trusted oracle. This work shows that an attacker who can _act in the environment_ — not steal weights, not corrupt logs, just participate — is sufficient to degrade learning by 18–98% depending on algorithm and setting.

Two practical takeaways:

- **PPO needs a clip-budget defense.** On-policy clipping is a feature, but it's also what locks the policy into a poisoned basin. Detecting trap stickiness (e.g., monitoring KL between lure-region and global policy) is a near-term defense.
- **SAC's robustness is buffer-size dependent.** Multi-agent SAC's resilience comes from sample dilution; tune replay sizes against expected attacker throughput, or the single-agent collapse mode reappears.

## What's next

- **Trap-region detectors** — auxiliary monitors that flag clip-bounded KL collapse around suspicious reward clusters.
- **Robust reward estimation** — separating per-agent intrinsic reward streams so a single corrupted channel can't dominate the shared signal.
- **Real-robot transfer** — extending from the Unity benchmark to physical fleet scenarios (taxi ride-sharing dispatchers, multi-robot warehouses).

<hr/>

## BibTeX {#bibtex}

```bibtex
@inproceedings{choi2025poisoning,
  title     = {Poisoning Attacks on Multi-Agent Reinforcement Learning Systems},
  author    = {Choi, Chanyeok and Cho, Jaehwan and Lee, Youngmoon},
  booktitle = {IEEE-RAS International Conference on Humanoid Robots (Humanoids), Late-Breaking Report},
  year      = {2025},
}
```
