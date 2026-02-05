---
layout: project
title: "SPH Fluid Simulator"
subtitle: "An efficienc CPU based implementatio of SPH"
date: 2024-06-15
tech_stack: [C++, VTK, CompactNSearch]
github: https://github.com/hanusspi/BasicPressureSolverLab
demo: null
image: https://via.placeholder.com/600x300/1E90FF/FFFFFF?text=Project+2
featured: true
---

## Overview

Implementation of a full SPH pressuer solver, using WCSPH and PBF, as well as an explicit viscossity, cohesion and adhesion model. For improves scene setup, the setting can be defined in a scene file, specifying the simulation parameters. Core project was build as part of a uni project. 

## Key Features

- Implementation of the explicit WCSPH and implicit PBF Pressure solver
- Addition of emitters, adhesion and cohesion model
- Sampling of surfaces with boundary particles and export of the particles to vtks
- Full surface reconstruction with mesh smoothing using Marching Cubes

## Technical Implementation

- Use of [CompactNSearch](https://github.com/InteractiveComputerGraphics/CompactNSearch) for fast and efficient neighborhood searches
- Levaraging OMP and SOAs for blazing fast iterations and quickest possible computations
- Use of boundary particles for boundary interactions
- Implementation of Marching Cubes for surface reconstruction

## Results & Learnings

- Enhanced understanding of lagrangian discretization using SPH
- Actual understanding of how pressure solvers work and which parameters matter
- Importance of performance oriented implementation, to reduce computation time

## Demo

Renders of the results (generated using Blender):

  <figure style="text-align: center;">
  <img src="img\posts\2025-09-25-Meshgeneration\DamBreak.mp4" alt="Sparse Hash Grid"
  style="max-width: 80%;">
  <figcaption style="font-style: italic; color: #666; margin-top: 5px;">Single Dam Break Scene</figcaption>
  </figure>

  <figure style="text-align: center;">
  <img src="img\posts\2025-06-13-Fluid-Simulation2\adhesion.mp4" alt="Sparse Hash Grid"
  style="max-width: 80%;">
  <figcaption style="font-style: italic; color: #666; margin-top: 5px;">Effects of Adhesion and Cohesion</figcaption>
  </figure>


