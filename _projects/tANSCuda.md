---
layout: project
title: "Massive Parallel Entropy Encoding for GPUs"
subtitle: "Bringing ANS to the GPU"
date: 2025-01-01
tech_stack: [Cuda, C++]
github: https://github.com/hanusspi/Massively-Parallelized-ANS-Entropy-Encoder
demo: null
image: https://via.placeholder.com/600x300/0085A1/FFFFFF?text=Project+1
featured: true
---

## Overview
The project is a rework of my Bachelorthesis, turning the popular, but inherently sequential ANS algorithm into an efficient GPU applicable parallelized encoder. In the current form, it is a pure academic example, generating its own testdata, without any further application.

## Key Features

- **Feature 1**: Cuda implementation of parallized ANS
- **Feature 2**: Excessive testing framework for time and compressino performance

## Technical Implementation

- Design of a random data source, being able to handle very skewed symbol distributions, using geometric, zipf and custom symbol distributions
- Design of a new algorithm, that splits a single message into multiple messages per gpu thread. Each submessage is encoded and decoded in parallel on each Warp, leveraging Waveintrinsics (Direct3d), now transformed to Warp intrinsics
- Validation and correct en- and decoding of the messages were tested rigorosly, as well as the effects of message splitting on the compression performance

## Results & Learnings

- Deep dive into information theory
- Understanding and abstraction of ANS algorithm
- Opimazationn of algorithms for SIMD devices, with focus on memory coalescing and minimizing path divergence


