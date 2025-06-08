---
layout: post
title: "Introduction to SPH fluid simulation"
subtitle: "A first look at the physical framework."
background: '/img/posts/2023-08-13-Introduction-to-ANS/header.jpg'
---
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.9.1/chart.min.js"></script>

# 1. Introduction to Fluid Simulation

One of the more interesting topics in physics-based animation is the simulation of water and fluids in general. The relevance of this topic reaches from computer games, like Sea of Thieves, where a real-time simulation of the ocean is relevant, over VFX art for movies, like Avatar: The Way of Water, to many industrial applications like casting steel.

While there is a lot of research being done on this topic, I first came into contact with it during a physics-based animations lecture and have since worked hard on getting a deeper insight into the topic. Here I want to begin by introducing the fundamentals and then take a dive into some deeper topics that I am currently interested in.

There exist two (and a half) approaches for fluid simulation: the Lagrangian particle-based one and the Eulerian grid-based one. Furthermore, there is a lot of research going into hybrid models that try to combine the best out of both worlds. For now, I will focus on the Lagrangian approach.

# 2. Navier-Stokes Equation

First, let's think about what characteristics make a fluid a fluid and especially let's think about water. First, of course, it can flow freely and will always try to find a position of lowest energy. Furthermore, it is not compressible. So, (to an extent) no matter how deep you dive into the ocean, 1 cubic meter of water will always have the same amount of water molecules, no matter how high the pressure is. (Very much unlike air, which is very compressible) Finally, we have some more specific properties like surface tension.

The fundamental aspect of fluid simulation though is maintaining the incompressibility through time. And while this post will have a few formulas, I strive to make it as easy and interesting to understand as possible. And I promise in future posts, things will get more programming-centric. The incompressibility can simply be described by $$\frac{D \rho}{D t} = 0 \Leftrightarrow \nabla \cdot v = 0$$. $\rho$ denotes the density. The second part implies the same, but we will not use it any further.

From this we can derive the Navier-Stokes equation:
$$ \frac{D\mathbf{v}}{Dt}=-\frac{1}{\rho}\nabla p+\nu\nabla^2\mathbf{v}+\frac{\mathbf{f}}{\rho} $$
First of all, we are describing an acceleration. This acceleration has three components:

1. $$-\frac{1}{\rho}\nabla p$$: Acceleration due to pressure differences in the fluid.

2. $$\nu\nabla^2\mathbf{v}$$: Acceleration due to friction forces between particles in the fluid. This is generally known as viscosity.

3. $$\frac{\mathbf{f}}{\rho}$$: Accelerations due to external forces. The forces are normalized to a unit volume. The major force being used is gravity, but it can also be used to induce waves.

# 3. Smoothed Particle Hydrodynamics (SPH)

Currently, we managed to describe the properties of what we want to simulate with a formula. This is a good beginning, but the formula describes something temporally and spatially continuous, which we cannot compute. Therefore, we introduce a spatial discretization into finitely many particles. These particles are not really particles (but we call them that for simplicity) but more like probes giving us information about important field values like density and velocity at a position. This form of discretization makes it a Lagrangian viewpoint.

Having done the first step, we will use a trick to use this for discretization using the Dirac-Delta Distribution. This distribution has the simple characteristic that it is always zero, except at the point x where it jumps to infinity. Formally this looks like:

$$\delta(x)=\begin{cases}\infty&\mathrm{if~}x=0 \\ 0&\mathrm{otherwise}&\end{cases}$$

$$\int_{-\infty}^{+\infty}\delta(x)dx=1$$

Given a function $$f:\Omega\to\mathbb{R}^n$$ that maps a position vector $$x$$ in the domain $$\Omega \subset \mathbb{R}^3$$ to a scalar or vector value, it can be rewritten using the Dirac delta identity:

$$f(x)=\int_\Omega f(x^*)\delta(x-x^*)dx^*$$

While this does not really help us yet to determine somehow what values/properties our particles have, we can now simply rewrite the Dirac Delta identity using a kernel function.

$$f(\mathbf{x})=\int_\Omega f(\mathbf{x}^*)\delta(\mathbf{x}-\mathbf{x}^*)d\mathbf{x}^*$$

$$\approx\int_{D_{\mathbf{x}}}f(\mathbf{x}^*)W(\mathbf{x}-\mathbf{x}^*,h)d\mathbf{x}^*$$

Our kernel function will have to fulfill a few requirements. As we saw, the Dirac delta identity sums up to 1. This must be the case for the kernel function as well. Furthermore, it needs to be symmetric, positive, and have a compact condition. This simply means that it only returns a positive value as long as we are in a set radius, and otherwise we are just 0. For now, we will use the cubic spline kernel, which looks like this:

$$W(q) = \alpha \begin{cases}
\frac{2}{3} - q^2 + \frac{1}{2}q^3 & \text{if } 0 \leq q < 1 \\
\frac{1}{6}(2-q)^3 & \text{if } 1 \leq q < 2 \\
0 & \text{if } q \geq 2
\end{cases}$$

where $$q = \frac{\|\mathbf{x}_i - \mathbf{x}_j\|}{h}$$ and $\alpha$ is the normalization constant depending on in how many dimensions we run the simulation.

Taking a closer look at the relationship between the kernel function and the Dirac Delta identity:

<canvas id="sph-kernels-chart" width="700" height="500"></canvas>
<script>
(function() {
    // Cubic spline kernel (different smoothing lengths)
    function cubicSpline(r, h) {
        const q = Math.abs(r) / h;
        const sigma = 1 / h; // normalization constant (1D)
        
        if (q >= 2) return 0;
        if (q < 1) return sigma * (1 - 1.5 * q * q + 0.75 * q * q * q);
        return sigma * 0.25 * (2 - q) * (2 - q) * (2 - q);
    }
    
    // X values from -2 to 2
    const x = [];
    for (let i = -2; i <= 2; i += 0.02) {
        x.push(i);
    }
    
    // Different kernel smoothing lengths
    const kernels = [
        { h: 0.3, color: 'orange', label: 'h=0.3' },
        { h: 0.5, color: 'lightgreen', label: 'h=0.5' },
        { h: 0.7, color: 'cyan', label: 'h=0.7' },
        { h: 0.9, color: 'blue', label: 'h=0.9' },
        { h: 1.1, color: 'purple', label: 'h=1.1' }
    ];
    
    const datasets = kernels.map(kernel => ({
        label: kernel.label,
        data: x.map(xi => ({ x: xi, y: cubicSpline(xi, kernel.h) })),
        borderColor: kernel.color,
        backgroundColor: 'transparent',
        borderWidth: 3,
        pointRadius: 0,
        tension: 0
    }));
    
    // Add Dirac delta approximation (arrow pointing up)
    datasets.push({
        label: 'δ(x)',
        data: [{ x: 0, y: 0 }, { x: 0, y: 12 }],
        borderColor: 'red',
        backgroundColor: 'transparent',
        borderWidth: 4,
        pointRadius: 0,
        showLine: true
    });
    
    // Add arrowhead for Dirac delta
    datasets.push({
        label: '',
        data: [{ x: -0.05, y: 11.5 }, { x: 0, y: 12 }, { x: 0.05, y: 11.5 }],
        borderColor: 'red',
        backgroundColor: 'red',
        borderWidth: 3,
        pointRadius: 0,
        showLine: true,
        fill: true
    });
    
    const ctx = document.getElementById('sph-kernels-chart').getContext('2d');
    new Chart(ctx, {
        type: 'line',
        data: { datasets },
        options: {
            responsive: true,
            scales: {
                x: {
                    type: 'linear',
                    min: -2,
                    max: 2,
                    ticks: { stepSize: 0.5 },
                    title: { display: true, text: 'Distance (r/h)', font: { size: 14 } }
                },
                y: {
                    min: 0,
                    max: 13,
                    ticks: { stepSize: 2 },
                    title: { display: true, text: 'Kernel Weight W(r,h)', font: { size: 14 } }
                }
            },
            plugins: {
                legend: { 
                    display: true,
                    position: 'top',
                    labels: { filter: function(item) { return item.text !== ''; } }
                },
                title: {
                    display: true,
                    text: 'SPH Cubic Spline Kernels vs Dirac Delta Function',
                    font: { size: 16 }
                }
            }
        }
    });
    
    // Add text annotation for Dirac delta
    setTimeout(() => {
        const canvas = document.getElementById('sph-kernels-chart');
        const ctx = canvas.getContext('2d');
        ctx.fillStyle = 'red';
        ctx.font = '14px Arial';
        ctx.textAlign = 'center';
        ctx.fillText('δ(x)', canvas.width/2, 50);
        ctx.fillText('(h→0)', canvas.width/2, 65);
    }, 100);
})();
</script>

It becomes more obvious to see how an increasingly smaller h pulls the kernel function to a Dirac delta identity. Changing the h will later also influence how our simulation behaves. If the h is too small, it will explode; if it is too big, it will be very lazy, since there will be too much averaging between particles.

Finally, we can put it together and rewrite the continuous function into a discrete sum:

$$f(\mathbf{x}) \approx \sum_{j} V_j f(\mathbf{x}_j) W(\mathbf{x} - \mathbf{x}_j, h)$$

Now the quantity of f can be simply determined by a sum over its neighborhood in a fixed radius. Doing the neighborhood search in a naive brute force way is very expensive and scales horribly ($$O(n^3)$$). But luckily, there are some smarter ways to handle this. We will take a look into this at a later point, but for the beginning, we will use a library that does this for us. Using some smart math, we can now express the density of a particle using:

$$\rho_i = \sum_{j} \frac{m_j}{\rho_j} \rho_j W_{ij} = \sum_{j} m_j W_{ij}$$

What this means is that the closer the neighbors (the particle itself being the closest neighbor) are, the higher the influence of the density will be, which is exactly what we want. Many particles clustered together should result in a high density and sparsely seeded ones into a low one. This is visualized here:

<div style="text-align: center; margin: 20px; color: #666;">
    <h3>SPH Particle Influence Visualization</h3>
    <p>Click and drag to move the central particle. <strong>Blue circle:</strong> smoothing length (h), <strong>Red circle:</strong> support radius (2h)</p>
</div>
<canvas id="sph-influence-chart" width="600" height="600" style="border: 1px solid #ccc; display: block; margin: 20px auto;"></canvas>
<div style="text-align: center; margin: 20px;">
    <label>Smoothing Length (h): <input type="range" id="smoothing" min="30" max="120" value="80"></label>
    <span id="hValue">80</span>
    <div style="margin-top: 10px; font-size: 14px;">
        <span style="color: #0066cc;">● Smoothing Length (h)</span> | 
        <span style="color: #cc3300;">● Support Radius (2h)</span>
    </div>
</div>
<script>
(function() {
    const canvas = document.getElementById('sph-influence-chart');
    const ctx = canvas.getContext('2d');
    const smoothingSlider = document.getElementById('smoothing');
    const hValue = document.getElementById('hValue');
    
    let h = 80; // smoothing length
    let centerX = 300, centerY = 300;
    let isDragging = false;
    
    // Generate random particle positions
    const particles = [];
    for (let i = 0; i < 150; i++) {
        particles.push({
            x: Math.random() * 560 + 20,
            y: Math.random() * 560 + 20
        });
    }
    
    // SPH kernel function (cubic spline)
    function kernelWeight(r, h) {
        const q = r / h;
        if (q >= 2) return 0;
        if (q < 1) return 1 - 1.5 * q * q + 0.75 * q * q * q;
        return 0.25 * (2 - q) * (2 - q) * (2 - q);
    }
    
    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        // Draw influence field (gradient from center to support radius)
        const gradient = ctx.createRadialGradient(centerX, centerY, 0, centerX, centerY, 2 * h);
        gradient.addColorStop(0, 'rgba(255, 100, 100, 0.3)');
        gradient.addColorStop(0.5, 'rgba(255, 150, 150, 0.15)');
        gradient.addColorStop(1, 'rgba(255, 200, 200, 0.05)');
        
        ctx.fillStyle = gradient;
        ctx.beginPath();
        ctx.arc(centerX, centerY, 2 * h, 0, Math.PI * 2);
        ctx.fill();
        
        // Draw support radius (2h) - outer boundary
        ctx.strokeStyle = '#cc3300';
        ctx.lineWidth = 3;
        ctx.setLineDash([8, 4]);
        ctx.beginPath();
        ctx.arc(centerX, centerY, 2 * h, 0, Math.PI * 2);
        ctx.stroke();
        
        // Draw smoothing length (h) - inner reference
        ctx.strokeStyle = '#0066cc';
        ctx.lineWidth = 2;
        ctx.setLineDash([4, 4]);
        ctx.beginPath();
        ctx.arc(centerX, centerY, h, 0, Math.PI * 2);
        ctx.stroke();
        ctx.setLineDash([]);
        
        // Draw neighbor particles
        particles.forEach(particle => {
            const dx = particle.x - centerX;
            const dy = particle.y - centerY;
            const distance = Math.sqrt(dx * dx + dy * dy);
            const weight = kernelWeight(distance, h);
            
            // Color based on influence
            if (weight > 0) {
                const intensity = weight;
                ctx.fillStyle = `rgba(0, 100, 150, ${0.3 + 0.7 * intensity})`;
                ctx.strokeStyle = `rgba(0, 50, 100, ${0.5 + 0.5 * intensity})`;
                ctx.lineWidth = 2;
            } else {
                ctx.fillStyle = 'rgba(100, 150, 180, 0.2)';
                ctx.strokeStyle = 'rgba(70, 120, 150, 0.3)';
                ctx.lineWidth = 1;
            }
            
            ctx.beginPath();
            ctx.arc(particle.x, particle.y, 6, 0, Math.PI * 2);
            ctx.fill();
            ctx.stroke();
        });
        
        // Draw central particle
        ctx.fillStyle = '#ff3030';
        ctx.strokeStyle = '#cc0000';
        ctx.lineWidth = 3;
        ctx.beginPath();
        ctx.arc(centerX, centerY, 10, 0, Math.PI * 2);
        ctx.fill();
        ctx.stroke();
        
        // Draw cross on central particle
        ctx.strokeStyle = '#800000';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.moveTo(centerX - 6, centerY);
        ctx.lineTo(centerX + 6, centerY);
        ctx.moveTo(centerX, centerY - 6);
        ctx.lineTo(centerX, centerY + 6);
        ctx.stroke();
        
        // Add labels
        ctx.fillStyle = '#0066cc';
        ctx.font = '14px Arial';
        ctx.fillText('h', centerX + h + 10, centerY - 5);
        
        ctx.fillStyle = '#cc3300';
        ctx.fillText('2h', centerX + 2 * h + 10, centerY - 5);
    }
    
    // Mouse events
    canvas.addEventListener('mousedown', (e) => {
        const rect = canvas.getBoundingClientRect();
        const x = e.clientX - rect.left;
        const y = e.clientY - rect.top;
        const dx = x - centerX;
        const dy = y - centerY;
        if (dx * dx + dy * dy < 100) isDragging = true;
    });
    
    canvas.addEventListener('mousemove', (e) => {
        if (isDragging) {
            const rect = canvas.getBoundingClientRect();
            centerX = e.clientX - rect.left;
            centerY = e.clientY - rect.top;
            draw();
        }
    });
    
    canvas.addEventListener('mouseup', () => isDragging = false);
    
    // Smoothing length control
    smoothingSlider.addEventListener('input', (e) => {
        h = parseInt(e.target.value);
        hValue.textContent = h;
        draw();
    });
    
    draw();
})();
</script>

With this, we did the first big logical step into understanding fluid simulation.

# 4. Equation of State

Knowing the density of all our particles in the system now enables us to enforce the continuity equation. If the particle density is too high, this can be directly translated into a high pressure. Pressure forces are the forces that counteract high densities. Furthermore, taking the gradient of the density into account, we can determine in which direction the pressure is forcing our particle to retain constant density.

A mathematical formulation of this is presented in the equation of state (EOS):

$$p_i = \frac{\kappa \rho_0}{\gamma} \left( \left( \frac{\rho_i}{\rho_0} \right)^{\gamma} - 1 \right)$$

where the pressure $$p$$ of the particle $i$ is dependent on the pressure difference between the rest density $$\rho_0$$, which is the natural density of water and therefore for us $$1000kg/m^3$$, and the actual density of the particle. Furthermore, $$\gamma, \kappa$$ are stiffness parameters. For now, we will simply set $$\kappa$$ to 1, giving us $$p_i = \kappa (\rho_i - \rho_0)$$. Formulating it like this shows great similarity to a spring force, just defined as a density deviation times a stiffness constant.

While this gives us the pressure, later on we will also need the direction of the pressure. Therefore, it is noteworthy to quickly talk about the derivative of our kernel and density calculation. Mathematically, the derivative keeps the factors the same and just derives the kernel function itself. This derivative does not preserve linear and angular momentum though, since it is not symmetric. To maintain symmetry and satisfy Newton's laws, we therefore rephrase it as:

$$\nabla A_i \approx \rho_i \sum_{j} m_j \left( \frac{A_i}{\rho_i^2} + \frac{A_j}{\rho_j^2} \right) \nabla W_{ij}$$

# 5. Summary

With this, we already did the first steps into understanding the fundamentals of fluid simulation. First, we took a look into the general physical principles behind fluids and then derived a way to make it more computable. For the next part, we will take a look at a few more building blocks before we can start building a first fluid simulator.