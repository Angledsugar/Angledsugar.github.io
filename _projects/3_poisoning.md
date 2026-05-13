---
layout: page
title: Poisoning Attacks on Multi-Agent Reinforcement Learning Systems
description: Humanoids 2025 (Late-Breaking Report).
img: assets/img/7.jpg
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
  </div>
</div>

<hr/>

## TL;DR

We study **reward-poisoning adversarial attacks** on cooperative multi-agent reinforcement learning (MARL) systems — including controllers for taxi ride-sharing fleets — and characterize the attack budgets under which policy degradation becomes observable. The result: small, well-targeted reward perturbations can steer a converged cooperative policy toward systematically degraded behavior without tripping the obvious anomaly detectors.

## Why this matters

Cooperative MARL is increasingly used to coordinate fleets — ride-sharing dispatchers, warehouse robots, drone swarms. These systems share a critical assumption: the reward signal each agent observes is **trusted**. In practice, the reward comes from sensors, billing systems, or service-level metrics — all of which can be tampered with by an adversary who controls part of the data pipeline.

If a small fraction of reward signal is corrupted, can the attacker steer the global policy? **Yes**, and the budget required is smaller than the naive bound suggests, because cooperative policies amplify perturbations through their shared value function.

## Setting

- **Environment**: cooperative MARL with shared reward, including a taxi ride-sharing dispatcher with N agents.
- **Threat model**: attacker can perturb a fraction ε of the reward observations at training time. No access to policy weights, no access to other agents' observations.
- **Objective**: characterize the relationship between (ε, attack horizon, observed return) and quantify when the degradation becomes statistically detectable vs. when it stays hidden.

## What we find

- A **non-trivial attack budget threshold** exists: below ε*, the policy degrades but the degradation is indistinguishable from training noise; above ε*, the policy collapses visibly.
- The threshold scales with the **number of cooperating agents** — more agents means _easier_ to attack, because the shared value function spreads each perturbation across more updates.
- For taxi ride-sharing specifically, the attack manifests as a learned bias toward unprofitable areas, which is consistent with the corrupted reward signal but completely invisible to per-trip auditing.

## What's next

- Extending from reward poisoning to **observation poisoning** (adversarial perturbations on per-agent state inputs).
- Defenses: robust reward estimators that detect ε\* before the policy collapses.

<hr/>

## BibTeX

```bibtex
@inproceedings{choi2025poisoning,
  title     = {Poisoning Attacks on Multi-Agent Reinforcement Learning Systems},
  author    = {Choi, Chanyeok and Cho, Jaehwan and Lee, Youngmoon},
  booktitle = {IEEE-RAS International Conference on Humanoid Robots (Humanoids), Late-Breaking Report},
  year      = {2025},
}
```
