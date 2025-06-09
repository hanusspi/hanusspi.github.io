---
layout: post
title: "Our first SPH Solver"
subtitle: "Concept and Implementation of WCSPH."
background: '/img/posts/2023-08-13-Introduction-to-ANS/header.jpg'
---
# 1. Introduction

In the previous chapters I talked alot about physics and maths building up some fundamental ideas of why and how the lagrangian viewpoint can  be applied to fluid simulation. Now it is time, to turn the formulas into an algorithm and code to start some fluid simulation. The algorightms we use to simulate fluid, are generally called pressure solvers, since that is exactly what they do. 

# 2. WCSPH

There is many ways to implement an SPH Solver, but one of the oldest and simplest (and easiest to understand) is Weakly Compressible SPH.

Let me start by showing you the general algorithm and then jump into the details and some practically relevant side notes. 

{% highlight python linenos %}
FOR each particle i:
    find_neighbors(i) → neighbors_j

FOR each particle i:
    density_i = SUM(mass_j * kernel_weight(i,j)) for all neighbors j
    pressure_i = state_equation(density_i)

FOR each particle i:
    pressure_force_i = -(mass_i/density_i) * gradient(pressure_i)
    viscosity_force_i = mass_i * viscosity * laplacian(velocity_i)
    other_force_i = mass_i * gravity
    
    total_force_i = pressure_force_i + viscosity_force_i + other_force_i

FOR each particle i:
    velocity_i(t + dt) = velocity_i(t) + dt * total_force_i / mass_i
    position_i(t + dt) = position_i(t) + dt 
{% endhighlight %}