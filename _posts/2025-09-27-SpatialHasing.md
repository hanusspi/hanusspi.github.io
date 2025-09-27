---
layout: post
title: "Neighborhood Search for Particle Systems"
subtitle: "Leveraging Spatial Acceleration Structures for Fluid Simulation."
background: '/img/posts/2023-08-13-Introduction-to-ANS/header.jpg'
---
# 1. Introduction
In the previous posts, we looked a lot into fluid simulation. One major aspect of fluid simulation is the neighborhood search. Previously we just glanced over the fact and solved the issue using a library. Today we want to explore one method in more detail to achieve a faster neighborhood search.

# 2. Neighborhood Search
For simplicity, let's say we want to create a simple particle simulation, where we spawn a bunch of particles in the beginning and just run collision detection between the particles and a bounding box.

First we need to detect collisions, which will be the main focus. Then later on we will talk about the collision resolution. From our collision we need two major pieces of information. How deep is the collision, so how big is the overlap and in which direction. So for particles, the direction is the difference of center coordinates of both particles.

Checking if a particle collides with an AABB (axis aligned bounding box) is easy, by comparing the x,y,z coordinates to the x,y,z coordinates of the two points defining the AABB. Therefore we just iterate over all particles, compare if the particle center is closer than the particle radius to the boundary and apply forces accordingly to resolve the collision.

The more complicated part is, finding inter particle collisions. First lets define the distance between two spheres a and b:
$$d = ||p_1 - p_2|| - (r_1 + r_2).$$
If the distance is smaller than 0, we have a collision, and already the value we need for resolution.

So one way to check, would be to simply compare all sphere positions to each other:
{% highlight text linenos %}
FOR (size_t i = 0; i < particles.size(); ++i) {
    FOR (size_t j = i + 1; j < particles.size(); ++j) {
        glm::vec3 direction = particles[j].position - particles[i].position;
        float distance = glm::length(direction);
        float minDistance = particles[i].radius + particles[j].radius;

        if (distance < minDistance) {
            float overlap = minDistance - distance;
            CollisionPair collision{
                i, j, overlap, glm::normalize(direction)
            };
            collisions.push_back(collision);
        }
    }
{% endhighlight %}
Since we have two nested for loops both with length `particles.size()` we have a runtime of $$O(n^2)$$ with n being the number of particles. For 100 particles this might be still ok, but for 1.000 particles we already need to run 1.000.000 comparisons and for 10.000 particles 100.000.000 comparisons. Considering that our basic fluid simulations ran with 20.000 particles already, we need to find a faster solution.

# 3. Spatial Hashing
A major component of accelerating the search is the use of spatial acceleration data structures. For one we could add a voxel grid to our domain with a voxel size of the particle radius. Now we denote which particle lives in which voxel cell. If we now want to know which particles might have a collision with particle a, we simply in 2D look at the cell that particle a is in and the 8 surrounding cells. For the 3D case we then have to search 27 cells. Therefore for each particle we have to only look at a fixed amount of cells. This approach is considered a dense grid representation. Let's take a look at an example:

  <figure style="text-align: center;">
  <img src="\img\posts\2025-09-27-SpatialHashing\HashingDenseGrid.png" alt="Dense Hash Grid"
  style="max-width: 80%;">
  <figcaption style="font-style: italic; color: #666; margin-top: 5px;">Dense Hash Grid</figcaption>
  </figure>

Logically this means that first we iterate over all particles, determine each particle's cell id and increment the count of particles for each cell. The cell id is the index in the cell array. For later access we then create an entry points array. At the index i, the entry points array tells us at which index in the particle array we have to look for the particles of the cells. And at the index i + 1 of the entry points array we get the index of the next starting particle for the next cell (and thereby the index of the last element as the predecessor).

Performing the neighborhood search now is easy, by determining the indices of the neighboring 26 cells, looking up the particles inside the cells and iterating over them. 

This approach has a few issues though. First it forces us to determine the domain size in the beginning, which limits the dynamic evolution of the scene. Second, most simulations have a high air to particle ratio, rendering many cells empty. And lastly, by design this approach works best if there is only a small amount of particles in a cell. Therefore the cells should be small. All factors combined can lead to very big grids with very few entries, being slow and memory hungry. 

To circumvent this, instead of calculating our index like in the marching cubes example, we introduce a hash function:

$$hash(coords) = |((coords.x * 73856093) ^ (coords.y * 19349663) ^ (coords.z * 83492791))| \mod numCells $$

First of all, now we can also see negative indices, since an integer coordinate is simply the world coordinate divided by the cell size. Therefore we need to force the result into a positive range. Next we can preselect how many cells we want to have. Choosing too many cells leads to many sparse entries. Choosing too few cells can lead to overlap. Two different integer coordinates can lead to the same index. To minimize this, literature suggests using large prime numbers. If the overlap is not too big, we will simply iterate over a few more entries.

  <figure style="text-align: center;">
  <img src="\img\posts\2025-09-27-SpatialHashing\HashingSparse.png" alt="Dense Hash Grid"
  style="max-width: 80%;">
  <figcaption style="font-style: italic; color: #666; margin-top: 5px;">Dense Hash Grid</figcaption>
  </figure>

Therefore the general process is very similar. One thing we didn't look at yet is how to determine the particle array and how to actually build the entry points array. The second image shows a simple way of creating the particle array, by first building up a partial sums array that simply adds up as the entries continue over the index. Next we fill in each particle into the particle array, with the index being determined by the subsequent partial sums entry and decrement the entry in the partial sums array. This leaves us with a particle array and a partial sums array.

Dependent on how often the neighborhood information needs to be accessed, the neighborhood information can also be cached in a neighbors array, as done in my code sample [https://github.com/hanusspi/BasicParticleSimulation](https://github.com/hanusspi/BasicParticleSimulation). 

Considering that this implementation is very poorly optimized and majorly created as a test bed for a bigger framework, we can still see drastic performance improvements growing with the number of particles. And most importantly, doubling the number of particles only halves instead of quarters the framerate now.

# 4. Summary

Today we analyzed one major bottleneck in particle based simulations and derived two new methods of improving the performance. Firstly we built up an easier system using dense grid spatial data structure and then improved performance even further by applying sparse hash grids. While this helps performance a lot, especially the fluid simulation still runs very slow, as it is highly serialized. One means to improve performance would be to increase clock rates of the CPU. But here we are very limited. And if we cannot scale in depth, we have to scale in width. While CPUs are still very limited in this department, GPUs are not. Therefore next time we will take a deeper look at leveraging GPUs to gain massive performance improvements.

