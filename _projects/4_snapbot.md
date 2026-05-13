---
layout: page
title: "Snapbot: Enabling Dynamic Human-Robot Interactions for Real-Time Computational Photography"
description: HRI 2024 (Late-Breaking Report).
img: assets/img/8.jpg
importance: 4
category: research
abbr: 2024
year: 2024
authors: Chanyeok Choi, Youngmoon Lee
related_publications: true
---

<div class="row">
  <div class="col-12 text-center">
    <p class="mb-1">
      <a href="/">Chanyeok Choi</a><sup>1*</sup>, <a href="https://sites.google.com/umich.edu/youngmoonlee/home">Youngmoon Lee</a><sup>1</sup>
    </p>
    <p class="text-muted mb-3" style="font-size: 0.9em;"><sup>1</sup> Hanyang University</p>
    <p class="mb-3">
      <span class="badge bg-primary" style="font-size: 0.95em;">HRI 2024 · Late-Breaking Report</span>
    </p>
  </div>
</div>

<hr/>

## TL;DR

**Snapbot** is a manipulator-based photography system that frames, composes, and captures stylized portraits in real time — treating the human subject as a **dynamic interaction partner** rather than a static target. The robot continuously updates its camera viewpoint as the subject moves, applies stylization on the fly, and timing-locks the shutter to compositional cues.

## Why this matters

Existing robotic photography demos either (a) wait for the human to hold still and then take a fixed shot, or (b) record continuously and post-process. Both miss the interactive feedback loop that makes a portrait *good*: the photographer suggesting a pose, the subject responding, the photographer re-framing. Snapbot is an attempt to put a robot inside that loop.

## What we built

- **Platform**: a 6-DoF manipulator with an end-effector-mounted camera, framed as a real-time vision-control system.
- **Composition policy**: a lightweight head-pose and rule-of-thirds estimator that scores candidate viewpoints and drives the manipulator toward the highest-scoring one.
- **Stylization**: an AnimeGAN filter pipeline that runs on the live preview, so the subject sees the stylized output as they pose.
- **Shutter timing**: triggers when the composition score crosses a threshold *and* the subject's head pose is stable for ≥ 200ms — captures the intentional pose, not the transition.

## What we showed at HRI 2024

A live demo of the system framing, composing, and capturing stylized portraits with the subject continuously moving. The contribution is the **interaction model**, not the imaging pipeline — making the manipulator a participant in the portrait session rather than a tripod with extra steps.

## Related work

This system grew out of the **AnimeGAN Filter Photographer** project (SW Talent Festival 2023, ICT President's Award), where the same composition + stylization stack ran under safety-RL constraints on a fixed-base arm. Snapbot generalizes that work to dynamic, free-moving subjects.

## What's next

- **Subject-driven framing**: let the human nudge the robot's framing via gesture or gaze rather than only by moving their head.
- **Multi-subject scenes**: group portraits with active recomposition as people enter/leave the frame.

<hr/>

## BibTeX

```bibtex
@inproceedings{choi2024snapbot,
  title     = {Snapbot: Enabling Dynamic Human-Robot Interactions for Real-Time Computational Photography},
  author    = {Choi, Chanyeok and Lee, Youngmoon},
  booktitle = {ACM/IEEE International Conference on Human-Robot Interaction (HRI), Late-Breaking Report},
  year      = {2024},
}
```
