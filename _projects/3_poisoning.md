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
redirect: https://angledsugar.github.io/MAPA
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

A **reward-poisoning attacker agent**, trained jointly inside a multi-agent RL system, can drop one high-reward _poison cube_ per crawler that drags a converged cooperative policy off-trajectory — without ever touching weights or other agents' observations. On a Unity 50×50 m crawler benchmark, the same attack drops cumulative reward by **18.7% (PPO)** and **20.9% (SAC)** in the multi-agent setting; in the single-agent setting SAC collapses entirely (from 1276 → 23.93). The asymmetry has a structural cause: PPO's on-policy clipping locks the policy onto whichever poison cube it samples first, while SAC's off-policy + maximum-entropy replay dilutes poison samples — except when the buffer is too small to outvote a persistent attacker.

<div class="row mt-4">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/poisoning/poisonkiller_setup.png" title="Crawler/attacker environment" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption" style="text-align: left;">
    <strong>Setup.</strong> 50×50 m Unity environment. <em>Left</em>: baseline — blue crawler agents (the standard ML-Agents Crawler) navigate toward green target boxes to maximize cumulative reward. <em>Right</em>: under attack — a single attacker agent places one red poison cube per crawler at arbitrary locations; a crawler that touches a red cube receives a deceptively high reward, redirecting its policy away from the true target.
</div>

## Reward Poisoning Attacker model

Two agent classes coexist in one environment with **predefined, fixed reward rules**:

<div class="table-responsive">
  <table class="table table-sm align-middle">
    <thead>
      <tr>
        <th scope="col">Agent</th>
        <th scope="col">Goal</th>
        <th scope="col">Reward structure</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td><strong>Crawler</strong></td>
        <td>Reach the green target, maximize cumulative reward.</td>
        <td>+1 on touching a green cube.</td>
      </tr>
      <tr>
        <td><strong>Attacker</strong></td>
        <td>Poison the crawler's reward — pull it onto a red cube instead of the green target.</td>
        <td>+1 when a crawler touches a red poison cube it placed.</td>
      </tr>
    </tbody>
  </table>
</div>

The attacker has no access to crawler weights, no privileged sensors, no offline corruption of training data. It interacts only through the environment, by placing poisoned reward points — the same channel any other agent uses. This is what makes the attack realistic: any adversary that can _participate_ in a shared MARL environment can poison it.

Concretely, a **single attacker poisons one cube per crawler**:

1. **One poison cube per crawler, random placement.** A single attacker agent spawns one red poison cube per crawler at an arbitrary position in the 50×50 m arena — one cube in the single-agent setting, _N_ cubes for _N_ crawlers in the multi-agent setting. The green target is left untouched: the crawler's true goal still exists; the poison cube simply competes with it.
2. **Indistinguishable reward.** A crawler that touches the red cube collects the same high reward it would earn from the real green target, so at the reward level the poison is indistinguishable from a legitimate goal.
3. **Learned placement.** The attacker is itself an RL agent: it earns +1 only when a crawler reaches its red poison cube instead of the green target. Over training it learns where to drop each cube to capture the crawlers' trajectories most reliably.

The single- vs multi-agent axis in our experiments is therefore a property of the **crawler population** — one crawler versus many — while the attacker remains a single agent throughout.

## Why PPO and SAC respond differently

The paper's headline result is that **PPO is structurally more vulnerable to reward poisoning in multi-agent settings**, despite SAC showing a marginally larger absolute drop in the multi-agent column. The reason is mechanical:

- **PPO** is on-policy with a clipped surrogate objective. Once the poison perturbs the rollout distribution, the next batch over-samples the poisoned region; the clip then constrains policy updates _around the current (poison-biased) policy_. The result is **self-reinforcing trap capture** — the policy can't take a large enough step to escape the basin.
- **SAC** is off-policy with maximum-entropy regularization. The replay buffer dilutes poisoned samples across thousands of clean transitions, and the entropy term forces continued exploration around any apparent optimum. Poison capture requires the attacker to flood the buffer faster than it cycles.

The single-agent SAC collapse (1276 → 23.93) is the exception that proves the rule: with one crawler and one attacker, the buffer fills slowly enough that even a small number of poison samples become the dominant signal.

## Results

Cumulative reward at 1M training steps. "Drop" is the crawler's relative reward loss; the attacker column shows the attacker's reward under the same attack run.

<div class="table-responsive">
  <table class="table table-sm align-middle">
    <thead>
      <tr>
        <th scope="col">Scenario</th>
        <th scope="col" class="text-end">Crawler (baseline)</th>
        <th scope="col" class="text-end">Crawler (attack)</th>
        <th scope="col" class="text-end">Drop</th>
        <th scope="col" class="text-end">Attacker (attack)</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Multi-Agent PPO</td>
        <td class="text-end">528.4</td>
        <td class="text-end"><strong>429.4</strong></td>
        <td class="text-end">−18.7%</td>
        <td class="text-end">−2.903</td>
      </tr>
      <tr>
        <td>Multi-Agent SAC</td>
        <td class="text-end">971.3</td>
        <td class="text-end"><strong>769.9</strong></td>
        <td class="text-end">−20.9%</td>
        <td class="text-end">−2.449</td>
      </tr>
      <tr>
        <td>Single-Agent PPO</td>
        <td class="text-end">647.5</td>
        <td class="text-end"><strong>302.5</strong></td>
        <td class="text-end">−53.3%</td>
        <td class="text-end">+1</td>
      </tr>
      <tr>
        <td>Single-Agent SAC</td>
        <td class="text-end">1276</td>
        <td class="text-end"><strong>23.93</strong></td>
        <td class="text-end">−98.1%</td>
        <td class="text-end">−31.43</td>
      </tr>
    </tbody>
  </table>
</div>

<div class="row mt-3">
    <div class="col-sm-10 offset-sm-1">
        {% include figure.liquid loading="eager" path="assets/img/projects/poisoning/poisoning_result.png" title="Average reward under baseline vs. attack" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption" style="text-align: left;">
    <strong>Reward curves.</strong> Average reward versus training steps for PPO and SAC in single-agent (top) and multi-agent (bottom) environments, with and without the attacker. Attack runs show larger variance and lower asymptotes; the gap is most pronounced for single-agent SAC, where the buffer is too small to wash out poison samples.
</div>

## What this means for deployed MARL

Cooperative MARL is at the core of human-interactive robotics — humanoid teams, robot taxis, drone fleets. All of them share the same exposed surface: a reward signal that comes from the environment, not from a trusted oracle. This work shows that an attacker who can _act in the environment_ — not steal weights, not corrupt logs, just participate — is sufficient to degrade learning by 18–98% depending on algorithm and setting.

Two practical takeaways:

- **PPO needs a clip-budget defense.** On-policy clipping is a feature, but it's also what locks the policy into a poisoned basin. Detecting trap stickiness (e.g., monitoring KL between the poisoned region and the global policy) is a near-term defense.
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
