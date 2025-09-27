---
layout: post
title: "Turning Particles into a Fluid"
subtitle: "Marching Cubes and Surface Reconstruction."
background: '/img/posts/2023-08-13-Introduction-to-ANS/header.jpg'
---
# 1. Introduction
In the last post we created our first pressure solver and got some pretty good results. Real fluids though do not really look like a bunch of spheres, but have a closed fluid surface. To resolve this issue, we define a function that can tell us how far a point in space is from the fluid surface. To do this we will define a signed distance function, that works with the lagrangian fluid simulation. Furthermore we need a strategy to create points, for which we can sample the distance to the surface and turn those points into a grid. And this is what we will start with.

# 2. Marching Cubes
To start off with marching cubes algorighm [LC87] we overlay the simulation domain with a voxel grid. The length of a voxel we will denote with $$l$$. Next we sample some function, in most cases a signed distance field (sdf) $$\Phi$$ at each corner point $$x_i$$ of each voxel.

<iframe srcdoc='<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Marching Squares Animation</title>
    <style>
        body {
            margin: 0;
            padding: 20px;
            background: white;
            color: black;
            font-family: "Courier New", monospace;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        canvas {
            border: 2px solid #ccc;
            background: white;
            border-radius: 8px;
        }

        .controls {
            margin: 20px;
            display: flex;
            gap: 10px;
        }

        button {
            padding: 10px 20px;
            background: #e94560;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-family: inherit;
        }

        button:hover {
            background: #c73650;
        }

        .info {
            max-width: 600px;
            text-align: center;
            margin: 20px;
            line-height: 1.6;
        }

        .step-info {
            background: #f0f0f0;
            padding: 15px;
            border-radius: 8px;
            margin: 10px 0;
            min-height: 50px;
            border: 1px solid #ddd;
        }
    </style>
</head>
<body>
    <h1>Marching Squares Algorithm</h1>

    <canvas id="canvas" width="600" height="600"></canvas>

    <div class="controls">
        <button onclick="startAnimation()">Start Animation</button>
        <button onclick="stepForward()">Next Step</button>
        <button onclick="stepBackward()">Previous Step</button>
        <button onclick="reset()">Reset</button>
    </div>

    <div class="step-info" id="stepInfo">
        Click "Start Animation" to see how marching squares works!
    </div>

    <div class="info">
        <p>This animation shows the marching squares algorithm step by step:</p>
        <p>1. First, we define a shape using a Signed Distance Function (SDF)<br>
        2. We overlay a grid and sample the SDF at each grid point<br>
        3. We find where the surface crosses grid edges (where SDF changes sign)<br>
        4. Finally, we connect these crossing points to form the contour</p>
    </div>

    <script>
        const canvas = document.getElementById("canvas");
        const ctx = canvas.getContext("2d");
        const stepInfo = document.getElementById("stepInfo");

        const width = canvas.width;
        const height = canvas.height;
        const gridSize = 40;
        const centerX = width / 2;
        const centerY = height / 2;
        const sphereRadius = 180;

        let animationStep = 0;
        let animationFrame = null;
        let isAnimating = false;

        // Simple sphere SDF
        function sphereSDF(x, y) {
            const dx = x - centerX;
            const dy = y - centerY;
            const distFromCenter = Math.sqrt(dx * dx + dy * dy);
            return distFromCenter - sphereRadius; // negative inside, positive outside
        }

        function interpolate(p1, p2, v1, v2) {
            if (Math.abs(v1 - v2) < 0.001) return p1;
            const t = -v1 / (v2 - v1);
            return {
                x: p1.x + t * (p2.x - p1.x),
                y: p1.y + t * (p2.y - p1.y)
            };
        }

        function drawBackground() {
            ctx.fillStyle = "white";
            ctx.fillRect(0, 0, width, height);
        }

        function drawSphere() {
            ctx.save();
            ctx.globalAlpha = 0.2;

            // Draw filled circle
            ctx.beginPath();
            ctx.arc(centerX, centerY, sphereRadius, 0, Math.PI * 2);
            ctx.fillStyle = "#4a90e2";
            ctx.fill();

            ctx.restore();
        }

        function drawGrid() {
            ctx.strokeStyle = "#ccc";
            ctx.lineWidth = 1;

            // Vertical lines
            for (let x = gridSize; x < width; x += gridSize) {
                ctx.beginPath();
                ctx.moveTo(x, 0);
                ctx.lineTo(x, height);
                ctx.stroke();
            }

            // Horizontal lines
            for (let y = gridSize; y < height; y += gridSize) {
                ctx.beginPath();
                ctx.moveTo(0, y);
                ctx.lineTo(width, y);
                ctx.stroke();
            }
        }

        function drawSDFValues() {
            ctx.font = "12px Courier New";
            ctx.textAlign = "center";

            for (let x = 0; x <= width; x += gridSize) {
                for (let y = 0; y <= height; y += gridSize) {
                    const sdf = sphereSDF(x, y);
                    const isInside = sdf < 0;

                    // Draw point
                    ctx.beginPath();
                    ctx.arc(x, y, 4, 0, Math.PI * 2);
                    ctx.fillStyle = isInside ? "#e94560" : "#4a90e2";
                    ctx.fill();

                    // Draw value
                    ctx.fillStyle = isInside ? "#c73650" : "#2c5aa0";
                    ctx.fillText(sdf.toFixed(0), x, y - 8);
                }
            }
        }

        function drawInterpolatedPoints() {
            const points = [];

            for (let x = 0; x < width; x += gridSize) {
                for (let y = 0; y < height; y += gridSize) {
                    if (x + gridSize <= width && y + gridSize <= height) {
                        const corners = [
                            {x: x, y: y, sdf: sphereSDF(x, y)},
                            {x: x + gridSize, y: y, sdf: sphereSDF(x + gridSize, y)},
                            {x: x + gridSize, y: y + gridSize, sdf: sphereSDF(x + gridSize, y + gridSize)},
                            {x: x, y: y + gridSize, sdf: sphereSDF(x, y + gridSize)}
                        ];

                        // Check each edge for zero crossings
                        for (let i = 0; i < 4; i++) {
                            const c1 = corners[i];
                            const c2 = corners[(i + 1) % 4];

                            if ((c1.sdf < 0) !== (c2.sdf < 0)) {
                                const point = interpolate(
                                    {x: c1.x, y: c1.y},
                                    {x: c2.x, y: c2.y},
                                    c1.sdf, c2.sdf
                                );
                                points.push(point);
                            }
                        }
                    }
                }
            }

            // Draw interpolated points
            ctx.fillStyle = "#ff8800";
            points.forEach(point => {
                ctx.beginPath();
                ctx.arc(point.x, point.y, 6, 0, Math.PI * 2);
                ctx.fill();
            });
        }

        function drawContour() {
            ctx.strokeStyle = "#00aa44";
            ctx.lineWidth = 3;

            for (let x = 0; x < width; x += gridSize) {
                for (let y = 0; y < height; y += gridSize) {
                    if (x + gridSize <= width && y + gridSize <= height) {
                        const corners = [
                            {x: x, y: y, sdf: sphereSDF(x, y)},
                            {x: x + gridSize, y: y, sdf: sphereSDF(x + gridSize, y)},
                            {x: x + gridSize, y: y + gridSize, sdf: sphereSDF(x + gridSize, y + gridSize)},
                            {x: x, y: y + gridSize, sdf: sphereSDF(x, y + gridSize)}
                        ];

                        const intersections = [];

                        // Find intersections on edges
                        for (let i = 0; i < 4; i++) {
                            const c1 = corners[i];
                            const c2 = corners[(i + 1) % 4];

                            if ((c1.sdf < 0) !== (c2.sdf < 0)) {
                                const point = interpolate(
                                    {x: c1.x, y: c1.y},
                                    {x: c2.x, y: c2.y},
                                    c1.sdf, c2.sdf
                                );
                                intersections.push(point);
                            }
                        }

                        // Draw line between intersections
                        if (intersections.length >= 2) {
                            ctx.beginPath();
                            ctx.moveTo(intersections[0].x, intersections[0].y);
                            ctx.lineTo(intersections[1].x, intersections[1].y);
                            ctx.stroke();
                        }
                    }
                }
            }
        }

        function drawStep() {
            drawBackground();

            switch (animationStep) {
                case 0:
                    stepInfo.textContent = "Step 1: Show the shape (sphere) we want to trace";
                    drawSphere();
                    break;

                case 1:
                    stepInfo.textContent = "Step 2: Overlay a grid";
                    drawSphere();
                    drawGrid();
                    break;

                case 2:
                    stepInfo.textContent = "Step 3: Sample SDF at grid points (red = inside, blue = outside)";
                    drawSphere();
                    drawGrid();
                    drawSDFValues();
                    break;

                case 3:
                    stepInfo.textContent = "Step 4: Find interpolated points where surface crosses edges (orange dots)";
                    drawGrid();
                    drawSDFValues();
                    drawInterpolatedPoints();
                    break;

                case 4:
                    stepInfo.textContent = "Step 5: Connect crossing points to form the final contour!";
                    drawGrid();
                    drawSDFValues();
                    drawInterpolatedPoints();
                    drawContour();
                    break;
            }
        }

        function animate() {
            if (!isAnimating) return;

            drawStep();

            if (animationStep < 4) {
                setTimeout(() => {
                    animationStep++;
                    animate();
                }, animationStep === 0 ? 2000 : animationStep < 2 ? 2000 : 3000);
            } else {
                isAnimating = false;
            }
        }

        function startAnimation() {
            animationStep = 0;
            isAnimating = true;
            animate();
        }

        function stepForward() {
            isAnimating = false;
            if (animationStep < 4) {
                animationStep++;
            }
            drawStep();
        }

        function stepBackward() {
            isAnimating = false;
            if (animationStep > 0) {
                animationStep--;
            }
            drawStep();
        }

        function reset() {
            animationStep = 0;
            isAnimating = false;
            drawBackground();
            stepInfo.textContent = "Click \"Start Animation\" to see how marching squares works! Or use \"Next Step\" for manual control.";
        }

        // Initialize
        reset();
    </script>
</body>
</html>' width="100%" height="800" frameborder="0"></iframe> 

The first major issue of this algorithm is indexing. We have a 3d voxel grid, but need to store everything in a 1d array. Therefore we determine unique integer coordinates (i,j,k) (which are easy to translate into world space) with (0,0,0) at the origin of the grid. Given the size of our simulation domain, which we can represent as an Axis Aligned Bounding Box (AABB) we can determine how many voxel cells the grid has in each dimension, denoted by $$n_x, n_y, n_z$$. Next we need to map each integer coordinate to a vertex (corner of a voxel) with

$$V = in_y n_z + j n_z + k$$.

In the same fashion a cell can be identified with

$$C = in_y n_z + j n_z + k$$.

Even though we have one less cell in each direction than vertices. But for simplicity we keep it like that.

And lastly we need a function to define edges

$$E = 3 V + dir,$$

where dir is 0,1,2 for x,y,z. 

For the full implementation we will have to handle some literal edge cases, because vertices on the outer side of the grid do not have edges in all directions.

The nice thing about those functions is, that they are bijective, therefore from a vertex id we can backtrack to its integer and finally world coordinates. Given the world coordinate x we can sample $$\phi(x)$$ and get for the vertex i $$\Phi_i = \Phi(x_i)$$.

With this information we know if a vertex is inside or outside of the surface. This already gives us some basic information, but not a mesh yet. 

Therefore next we will iterate over all edges and check if they intersect the surface, i.e. if the sign between the vertices making up the edge changes. If we have a sign change, we calculate an edge weight by linear interpolation giving us the intersection point. This gives us a solid guess of where the actual surface could be. It is important that we store these points in relation to the edge they belong to. Since our surface is going to have a lot fewer points than we have voxels, it is sensible to store them in a map. 

  <figure style="text-align: center;">
  <img src="/img/posts/2025-09-25-Meshgeneration/MarchingCubesEdit.svg.png" alt="Marching Cubes visualization"
  style="max-width: 80%;">
  <figcaption style="font-style: italic; color: #666; margin-top: 5px;">Source: Wikipedia</figcaption>
  </figure>

In the last step we need to append the vertex data with geometric connectivity. For this a marching cubes table exists, that we can use. As visible in the image above, the marching cube table tells us, based on which edges experiencing a sign change, which vertices to connect. And for this to work properly, all the previous hassle with indexing was required. 

Lastly, to get a smooth mesh, we do not just need to have vertices and connectivity, but also vertex normals. These we can either calculate in the beginning for the voxel corners and interpolate later on or calculate in a second sweep for the exact vertex positions. 

Applying this for an object with a known signed distance function, like a torus yields this result:

<figure style="text-align: center;">
<img src="/img/posts/2025-09-25-Meshgeneration/torus.png" alt="Torus generated using marching cubes" style="max-width: 80%;">
<figcaption style="font-style: italic; color: #666; margin-top: 5px;">Torus mesh generated using the marching cubes algorithm</figcaption>
</figure>

# 3. Fluid Surface Reconstruction

To make marching cubes work we therefore need a $$\Phi$$ that we can sample. While for primitive shapes, like a torus, sphere or square these are analytically defined, for our fluid they are not. Therefore we need a function that is close to 1, if it is inside the fluid and 0 or even negative if it is outside of the fluid. Applying some fundamentals of the SPH idea and using the method how we determine the value of quantites in SPH (as discusses in the last posts) gives us [BJ25]

$$\Phi(\mathbf{x})=-c+\sum_{j\in\mathcal{N}(\mathbf{x})}\frac{1}{\rho_j}W(\mathbf{x}-\mathbf{x}_j,h),$$

where $$\rho_j$$ is now the normalized density estimation

$$\rho_i=\sum_{j\in\mathcal{N}(\mathbf{x}_i)}W(\mathbf{x}_i-\mathbf{x}_j,h).$$

We are required to add a slightly negative c to force a sign change, since the second part of the function will never get smaller than 0. By choosing the c we can determine how tight or loose we want the surface to be. A good starting value is 0.55.

Since this formulation seemingly requires a neighborhood search, it seems to be pretty expensive. Since unlike the particles, our vertices are grid aligned and indexable though, we can turn things around and iterate over the particles, determine an AABB around each particle, check which vertices are in the interior of the AABB and then write the values to the vertices. The only disadvantage of this approach is that it is very hard to parallelize.

In the very beginning we defined the cell size as a parameter. We want to set the cell size so that a sole particle can create a single mesh, to prevent particles suddenly popping up and vanishing later on. Therefore a good cell size is 1.0 times the particle radius. 

# 4. Results

Applying marching cubes and the newly defined sdf gives us this lovely result for our previously generated simulation

<video width="70%" controls>
  <source src="/img/posts/2025-09-25-Meshgeneration/DamBreak.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

Depending on the rendering settings in Blender the look of the fluid can be varied even further.

# 5. Summary
In this post we started out by defining a voxel grid with many indices turning it into the marching cube algorithm [LC87]. Finally we managed to apply it to a particle based fluid simulation turning it into a real fluid.
The code can be found under [https://github.com/hanusspi/BasicPressureSolverLab](https://github.com/hanusspi/BasicPressureSolverLab), where the mesh gets generated as a vtk and can be imported into blender using the same workflow as for the particles in the previous blog post.

# References

[LC87] W.E. Lorensen and H.E. Cline. Marching cubes: A high resolution 3d surface construction algorithm. ACM Computer Graphics, 21(4):163–169, 1987.
[BJ25] Prof. Dr. Jan Bender, M.Sc. Timna B¨ottcher, M.Sc. Jos´e Fern´andez, M.Sc. Lukas Westhofen: Fluid Simulation in Computer Graphics