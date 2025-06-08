---
layout: post
title: "Introduction to SPH fluid simulation"
subtitle: "A first look at the physical framewirk."
background: '/img/posts/2023-08-13-Introduction-to-ANS/header.jpg'
---

# 1. Introduction to Fluid Simulation

One of the more interesting topics in physics based animation is the simulation of water and fluids in general. The relevance of this topic reaches from computer games, like sea of thieves, where a real time simulation of the ocean is relevant, over vfx art for movies, like Avatar Water World to many industrial applications like casting steel.

While there is alot of research being done on this topic, I first came into contact with it during a physics based animations lecture and since have worked hard on getting a deeper insight into the topic. Here I want to begin by introducing the fundamentals and then take a dive into some deeper topics, that i am currently interested in.

There exist two (and a half) approaches for fluid simulation: the lagrangian particle based one and the euleraian grid based one. Further more there is alot of research going into hyrid models that try to combine the best out of both worlds. For now I will focus on the larangian approach.

# 2. Navier Stokes Equation

First, lets think about what charakteristics make a fluid a fluid and especially lets think about water. First of course it can flow freely and will always try to find a postion of lowest energy. Furthermore it is not compressible. So (to an extent) no matter how deep you dive into the ocean, 1 cubic meter of water will always have the same amount of water molecules, no matter hoch high the pressure is. Finally we have some more specific properties like surface tension. 

The fundamental aspect of fluid simulation though is maintaining the incompressibility through time. And while this post will have a few formulas i strive to make it as easy and interesting to understand as possible. And I promise in future things will get more programming centric. The incompressibility can simply be described by $$\frac{D \rho}{D t} = 0 \Leftrightarrow \nabla \cdot v = 0$$. $\rho$ denotes the density. The second part implies the same, but we will not use any further. 

From this we can derive the navier stokes equation:
$$ \frac{D\mathbf{v}}{Dt}=-\frac{1}{\rho}\nabla p+\nu\nabla^2\mathbf{v}+\frac{\mathbf{f}}{\rho} $$
First of all we are describing an acceleration. This acceleration has three components.

1. $$-\frac{1}{\rho}\nabla p$$: Acceleration due to pressure differences in the fluid.

2. $$\nu\nabla^2\mathbf{v}$$: Acceleation due to firction forces between particls in the fluid. This is generally known as viscosity.

3. $$\frac{\mathbf{f}}{\rho}$$: Accelerations due to external forces. The forces are normalized to a unit volume. Major force being used is gravity, but can also be used to induce waves.

# 3. Smoothed Particle Hydrodynamics (SPH)

Currently we managed to describe the properties of what we want to simulate with a formula. This is a good beginning, but the formula describes something temporal and spacially continous, which we cannot compute. Therefore we inroduce a spatial discretization into finatly many particles. These particles are not really particles (but we call them that for simplicity) but more like probes giving us information about important field values like density and velocity at a position. This form of discretiation makes it a lagrangian viewpoint.

Having done the first step, we will use a trick to use this for discretization using the Dirac-Delta Distribution. This distribution has the simple characteristic, that it is always zero, except at the point x where it jumps to infinity. Formally this looks like:

$$\delta(x)=\begin{cases}\infty&\mathrm{if~}x=0 \\ 0&\mathrm{otherwise}&\end{cases}$$

$$\int_{-\infty}^{+\infty}\delta(x)dx=1$$

Given a fucntion $$f:\Omega\to\mathbb{R}^n$$ that maps a position vector $$x$$ in the domain $$\Omega \subset \mathbb{R}^3$$ to a scalar or vector vlaue, it can be rewritten using the Dirac delta identiy:

$$f(x)=\int_\Omega f(x^*)\delta(x-x^*)dx^*$$ text

While this does not really help us yet to determine somehow what values/properties our particles have, we can now simply rewrite the Dirac Delta identity using a kernel function.

$$f(\mathbf{x})=\int_\Omega f(\mathbf{x}^*)\delta(\mathbf{x}-\mathbf{x}^*)d\mathbf{x}^*$$

$$\approx\int_{D_{\mathbf{x}}}f(\mathbf{x}^*)W(\mathbf{x}-\mathbf{x}^*,h)d\mathbf{x}^*$$

The kernel approximation becomes:

$$f(\mathbf{x}) = \int_{\Omega} f(\mathbf{x}^*) \delta(\mathbf{x} - \mathbf{x}^*) \, d\mathbf{x}^*$$

$$\approx \int_{D_{\mathbf{x}}} f(\mathbf{x}^*) W(\mathbf{x} - \mathbf{x}^*, h) \, d\mathbf{x}^*$$

\frac{D\mathbf{v}}{Dt} &= -\frac{1}{\rho}\nabla p + \nu\nabla^2\mathbf{v} + \frac{\mathbf{f}}{\rho} \\
\text{where:} \quad &  \\
\frac{D\mathbf{v}}{Dt} &= \text{material derivative of velocity} \\
-\frac{1}{\rho}\nabla p &= \text{pressure gradient acceleration} \\
\nu\nabla^2\mathbf{v} &= \text{viscous acceleration} \\
\frac{\mathbf{f}}{\rho} &= \text{external force acceleration}
\end{align}$$

### SPH Kernel Approximation (Multi-line)
$$\begin{align}
f(\mathbf{x}) &= \int_{\Omega} f(\mathbf{x}^*) \delta(\mathbf{x} - \mathbf{x}^*) \, d\mathbf{x}^* \\
&\approx \int_{D_{\mathbf{x}}} f(\mathbf{x}^*) W(\mathbf{x} - \mathbf{x}^*, h) \, d\mathbf{x}^* \\
&\approx \sum_{j} \frac{m_j}{\rho_j} f(\mathbf{x}_j) W(\mathbf{x} - \mathbf{x}_j, h)
\end{align}$$

### Incompressibility Condition
$$\begin{align}
\frac{D\rho}{Dt} &= 0 \\
\Leftrightarrow \quad \nabla \cdot \mathbf{v} &= 0
\end{align}$$

### Kernel Function Properties
$$\begin{align}
\int_{\Omega} W(\mathbf{x} - \mathbf{x}^*, h) \, d\mathbf{x}^* &= 1 \quad \text{(normalization)} \\
\lim_{h \to 0} W(\mathbf{x} - \mathbf{x}^*, h) &= \delta(\mathbf{x} - \mathbf{x}^*) \quad \text{(delta function property)} \\
W(\mathbf{x} - \mathbf{x}^*, h) &= 0 \quad \text{for} \quad |\mathbf{x} - \mathbf{x}^*| > kh \quad \text{(compact support)}
\end{align}$$

## Single Line Examples

The density at position $\mathbf{x}$ is approximated as:
$$\rho(\mathbf{x}) = \sum_{j} m_j W(\mathbf{x} - \mathbf{x}_j, h)$$

The gradient of a field $f$ is:
$$\nabla f(\mathbf{x}) \approx \sum_{j} \frac{m_j}{\rho_j} f(\mathbf{x}_j) \nabla W(\mathbf{x} - \mathbf{x}_j, h)$$

## Advanced Formatting

### Piecewise Functions
$$W(r, h) = \frac{1}{\pi h^2} \begin{cases}
1 - \frac{3}{2}q^2 + \frac{3}{4}q^3 & \text{if } 0 \leq q < 1 \\
\frac{1}{4}(2-q)^3 & \text{if } 1 \leq q < 2 \\
0 & \text{if } q \geq 2
\end{cases}$$

where $q = \frac{r}{h}$.

### Matrix Notation
$$\begin{align}
\mathbf{A} = \begin{pmatrix}
\frac{\partial^2 p}{\partial x^2} & \frac{\partial^2 p}{\partial x \partial y} \\
\frac{\partial^2 p}{\partial y \partial x} & \frac{\partial^2 p}{\partial y^2}
\end{pmatrix}
\end{align}$$