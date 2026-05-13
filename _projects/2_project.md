---
layout: page
title: DATP
description: Delivery Any Thing Project — autonomous last-mile delivery platform (2019).
img: assets/img/3.jpg
importance: 2
category: hardware
related_publications: false
---

**DATP (Delivery Any Thing Project)** is an autonomous ground-delivery platform built for last-mile parcel transport in semi-structured environments — campus pathways, apartment complexes, and pedestrian zones.

## Why

Last-mile delivery is the most expensive segment of logistics, and the labor cost has been rising faster than parcel volume. Existing solutions split into two camps: drones (regulation-heavy, payload-limited) and bipedal/quadruped platforms (too expensive for routine deployment). DATP targets the middle — a wheeled platform with enough autonomy to handle pedestrians and curbs but cheap enough to deploy at scale.

## What we built

- **Platform**: 4-wheel differential drive base with shock-absorbing suspension; insulated cargo compartment with electronic lock.
- **Perception**: stereo camera + 2D LiDAR for obstacle detection and pedestrian tracking; GPS + wheel odometry for outdoor localization.
- **Navigation**: hybrid map-based / reactive planner — pre-computed corridor on known routes, local social-force model for pedestrian avoidance.
- **Handoff**: recipient-side mobile app to unlock the compartment via one-time code.

## What I learned

The hard part wasn't navigation — it was **predicting pedestrian intent**. A reactive avoidance policy is fine until two people approach from different angles and you have to choose who to yield to; humans do this with eye contact and we don't have a clean signal for it. We ended up biasing the planner to be conservative and just stop when ambiguity got high — slower but socially acceptable. The general lesson — that **knowing when not to act is half the problem** — is one of the threads that runs through my current research on critical-phase detection.

*Year:* 2019.
*Role:* Co-developer.
