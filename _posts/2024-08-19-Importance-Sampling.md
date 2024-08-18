---
layout: post
title: "Importance Sampling"
subtitle: "Enhancing Efficiency in Sampling"
background: '/img/posts/2023-08-13-Introduction-to-ANS/header.jpg'
---
# 1. Introduction to Importance Sampling

In the previous post, we explored Rejection Sampling as a method for sampling from complex distributions. While effective, rejection sampling can be inefficient, especially when a large number of samples are rejected. Importance Sampling offers an alternative approach that improves efficiency by assigning weights to samples instead of rejecting them.

Importance sampling is widely used in Bayesian inference, Monte Carlo simulations, and statistical estimation. In this post, we’ll dive into the concept of importance sampling, how it works, and play around with it in python.

# 2. The Idea behind Importance Sampling

For most use-cases the goal is not sampling from $$p(z)$$ by itself, but to evaluate expectations of the form

$$ \mathbb{E}[f] = \int f(z)p(z) dz $$

The exact solution of this integral is most of the time intractable, so we need to find a discrete solution. A simple strategy would be the use of grid sampling. For this we discretize $$z$$-space into a uniform grid. Now we evaluate the integrand as a sum of the form 

$$ \mathbb{E}[f] = \sum_{l=1}^L f(z^{(l)})p(z^{(l)})dz$$

For a low dimensional problem this is a good a solution, but the number of terms grows exponentially with number of dimensions.

But we can apply the same trick of rejection sampling and try to make the problem easier using a proposal distribution $$q(z)$$ from which it is easy to draw samples. Now we can use some trickes to reformulate the problem:

$$ \mathbb{E}[f] = \int f(z)p(z) dz = \int f(z)\frac{p(z)}{q(z)}q(z) dz $$ 

$$ \simeq \frac{1}{L} \sum_{l=1}^L \frac{p(z^{(l)})}{q(z^{(l)})} f(z^{(l)})$$ 

where $$ \frac{p(z^{(l)})}{q(z^{(l)})} $$ is considered the importance weight.

