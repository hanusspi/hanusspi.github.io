---
layout: post
title: "Turning Particles into a Fluid"
subtitle: "Marching Cubes and Surface Reconstruction."
background: '/img/posts/2023-08-13-Introduction-to-ANS/header.jpg'
---
# 1. Introduction
In the last post we created our first pressure solver and got some pretty good results. Real fluids though do not really look like a bunch of spheres, but have a closed fluid surface. To resolve this issue, we define a function, that can tell us how far a point in spcae is away from the fluid surface. To do this we will define a signed distance function, that works with the lagrangian fluid simulation. Further more we need a strategy, to create points, for which we can sample the distance to the surface and turn those points into a grid. And this is what we will start with.

# 2. Marching Cubes
To start of with marching cubes we overlay the simulation domain with a voxel grid. The length of a voxel we will denote with $$l$$. Next we sample some function, in most cases a signed distance field (sdf) $$\Phi$$ at each corner point $$x_i$$ of each voxel. 

The first major issue of this algorithm is indexing. We have a 3d voxel grid, but need to store everything in a 1d array. Therefore we determine unique integer coordinates (i,j,k) (which are easy to translate into world spcae) with (0,0,0) at the origin of the grid. Given the size of our simuatlion doamin, which we can represent as an Axis Aligned Bounding Box (AABB) we can determine how many voxel cells the grid has in each dimenstion, denoted by $$n_x, n_y, n_z$$. Next we need to map each integer coordinate to a vertex (corner of a voxel) with

$$V = in_y n_z + j n_z + k$$.

In the same fashion a cell can be identified with

$$C = in_y n_z + j n_z + k$$.

Even though we have one less cell in each direction then vertex. But for simplicity we keep it like that.

And lastly we need a function do define edges

$$E = 3 V + dir,$$

where dir is 0,1,2 for x,y,z. 

For the full implementatino we will have to handle some literal edge cases, because vertices on the outer side of the grid do not have edges in all directions.

The nice thing about those functions is, that they are bijective, therefore from a vertex id we can backtrack to its integer and finally world coordinates. Given the world coordinate x we can sample $$\phi(x)$$ and get for the verteix i $$\Phi_i = Phi(x_i)$$.

With this information we know if a vertex is inside or outside of the surface. This already gives us some basic information, but not a mesh yet. 

Therefore next we will iterate over all edges and check if they intersect the surface, therefore if the sign between the vertices making up the edge changes. If we have a sign change, we calculate an edge weight by linear interpolation giving us the intersection point. This gives us a solid guess of where the actual surface could be. It is important that we store these points in relation to the edge they belong too. since our surface is going to have alot less points then the we have voxels, it is sensible to store them in a map. 

![Imagetext](/img/posts/2025-09-25-Meshgeneration/MarchingCubesEdit.svg.png)

In the last step we need to append the vertex data with geometric connectivity. For this a marching cubes table exists, that we can use. As visible in the image above, the marching cube table tells us, based on which edges experiencing a sign change, which vertices to connect. And for this to work properly, all the previous hassle with indexin was required. 

Laslty, to get a smooth mesh, we do not just need to have vertices and connectivity, but also vertex normals. These we can either calculate in the beginning and for the voxel corners and interpolate later on or calcualte in a second sweep for the exact vertex positions. 

Applying this for an object with a know signed distance function, like a torus yields this result:

![Imagetext](/img/posts/2025-09-25-Meshgeneration/torus.svg.png)

# 3. Fluid Surface Reconstruction

# 4. Results

# 5. Summary