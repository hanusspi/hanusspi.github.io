---
layout: project
title: "CFD using Lattice Boltzman"
subtitle: "Exploring LBM using Cuda"
date: 2024-03-20
tech_stack: [C++, OpenGl, Cuda]
github: https://github.com/hanusspi/LatticeBoltzman
demo: null
image: https://via.placeholder.com/600x300/32CD32/FFFFFF?text=Project+3
featured: false
---

## Overview

Exploration of CDF methods for incompressible flow. For easiest GPU implementationm, Lattice Boltzman, using BGK in a 2d setting was implemented. The results are visualized as either a heatmap of velocity, or colored by flow direction.

## Key Features

- Interoperation between Cuda and OpenGl
- Exploration of the LBM method and the BGK operation
- Tested on lid dirven cavity and flow over a sphere, creating vorticity

## Technical Implementation

- Starting point is the cuda implementation of LBM
- 3d grid generation for the 2d LBM solver
- Collision, streaming and boundary step, exploring different boundary conditions
- Bufferswapping to reduce memory usage

## Results & Learnings

- High performance due to parallel implementation
- Parametertuning, and translation to grid units
- Challenges, visualizing the flow of a 2d flow field

## Documentation

Direct results of the solver:

<video width="100%" controls>
  <source src="img\projects\lbm\LidDrivenCavi.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

**Lid driven cavity, with colors indicating flow direction**


<video width="100%" controls>
  <source src="img\projects\lbm\KarmanVerti.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

**Flow over a sphere creating theKármán vortex street**

