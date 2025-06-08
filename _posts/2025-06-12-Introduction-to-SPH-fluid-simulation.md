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

Our kernel function will have to fullfill a few requirements. As we saw, the dirac delta identity sums up to 1. This must be the case for the kernel function as well. Further more it needs to be symmetric, positive and have a compact condition. This simply means, that it only returns a positive value, as long as we are in a set radius and elsewise we are just 0. For now we will use the cubic spline kernel, which looks like that:

$$W(q) = \alpha \begin{cases}
\frac{2}{3} - q^2 + \frac{1}{2}q^3 & \text{if } 0 \leq q < 1 \\
\frac{1}{6}(2-q)^3 & \text{if } 1 \leq q < 2 \\
0 & \text{if } q \geq 2
\end{cases}$$

where $q = \frac{\|\mathbf{x}_i - \mathbf{x}_j\|}{h}$ and $\alpha$ is the normalization constant depending on in how many dimensions we run the simulation. Finally we can put it together and rewrite the continous function into a discrete sum:

$$f(\mathbf{x}) \approx \sum_{j} V_j f(\mathbf{x}_j) W(\mathbf{x} - \mathbf{x}_j, h)$$

Now the quantity of f can be simply determined by a sum over its neighborhood in a fixed radius. Doing the neighborhoodsearch in a naiive brute force way is very expensive and scales horably ($$o(n^3)$). But luckily there is some smarter ways to handle this. We will take look into this at a later point, but for the begining we will use a library that does this for us. Using some smart maths, we can now express the density of a particle using:

$$\rho_i = \sum_{j} \frac{m_j}{\rho_j} \rho_j W_{ij} = \sum_{j} m_j W_{ij}$$

With this we did the first big logical step into understanding fluid simulation.

# 4 Equation of State

Knowing the density of all our particles in the system now enables us to enforce the continuity equation. If the particle density is to high, this can be directly translated into a high pressure. Pressure Forces are the forces that counteract high densities. Further more taking the gradient of the density into account, we can determine, in which direction the pressure is forcing our particle to retain constant density. 

A mathematical formulation of this is presented in the equation of state (EOS):

$$p_i = \frac{\kappa \rho_0}{\gamma} \left( \left( \frac{\rho_i}{\rho_0} \right)^{\gamma} - 1 \right)$$

where the pressure $p$ of the particle $i$ is dependent on the pressure difference between the rest density $\rho_0$ which is the natural density of water and therefore for us $1000kg/m^3$ and the actual density of the particle. Further more $\gamma, \kappa$ are stifness parameters. For now we will simply set $\kappa$ to 1 giving us $p_i = \kappa (\rho_i - \rho_0). Formulating it like this shows great similarity to a spring force, just defined as a density deviationt imes a stiffness constant.

While this gives us the pressure, later on we will also need the direction of the pressure. Therefore it is noteworth to quickly talk about the derivative of our kernel and density calculation. Mathematically the derivative keeps the factors the same and just derives the kernel function itself. This derivative does not preserves linear and angular momentum though, since it is not symmetric. To maintain symmetry and satisfy Newton, we therefore rephrase it as:

$$\nabla A_i \approx \rho_i \sum_{j} m_j \left( \frac{A_i}{\rho_i^2} + \frac{A_j}{\rho_j^2} \right) \nabla W_{ij}$$


# 5 Summary

With this we arleady did the first steps into understanding the fundamentals of fluid simulation. First we took a look into the general physical principal behind fluids and then derived a way to make it more computable. For the next part we will take a look at a few more building blocks, before we can start building a first fluid simulator.