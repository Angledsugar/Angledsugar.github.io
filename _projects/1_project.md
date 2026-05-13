---
layout: page
title: ACMR
description: Auto Climbing Mopping Robot — wall-climbing mobile platform with onboard mopping module (2020).
img: assets/img/12.jpg
importance: 1
category: hardware
related_publications: false
---

**ACMR (Auto Climbing Mopping Robot)** is a wall-climbing service robot designed to clean large vertical glass surfaces — building façades and tall interior windows — without scaffolding or human operators.

## Why

Façade cleaning is dangerous and labor-intensive: human cleaners on suspended cradles is still the dominant solution for tall buildings, and the accident rate is non-trivial. Existing window-cleaning robots either rely on permanent rail systems (expensive, building-specific) or vacuum/suction cups that fail on slightly uneven surfaces. ACMR targets the gap in between — a portable platform that can be deployed on standard glass with no building modification.

## What we built

- **Adhesion**: passive suction array with active pressure regulation, falling back to magnetic anchoring on framed sections.
- **Mobility**: 4-wheel skid-steer base tuned for vertical operation; tether-based safety line for fail-safe descent.
- **Cleaning module**: dual-tank (clean / dirty water) with a rotating microfiber pad; on-board pump.
- **Control**: closed-loop pose tracking via onboard IMU + tether-encoder odometry, autonomous boustrophedon coverage of the cleaning region.

## What I learned

A surprising amount of the engineering effort went into recovery behaviors — what to do when the suction breaks contact, what to do when the cleaning solution runs out mid-pass, what to do when the tether catches. The "happy path" demo was easy; making it field-deployable was 80% of the work. That intuition — that real-world robot deployments are bottlenecked by edge-case recovery rather than by nominal performance — is what eventually pushed me toward studying **failure detection in learned policies** (see my [CPD work](/publications/)).

*Year:* 2020.
*Role:* Engineering lead.
