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

![Imagetext](img/posts/2024-08-19-Importance-Sampling.md/importanceSamplingIdea.jpg)

Now we need to consider the typical setting, in which we can only evaluate $$p(z)$$ up to an unkown normalization constant $$p(z) = \tilde{p}(z)/Z_p$$. With $$q(z)$$ we can do the same: $$q(z) =\tilde{q}(z)/Z_q$$. From this we can construct:

$$ \mathbb{E}[f] = \int f(z)p(z) dz = \frac{Z_q}{Z_p} \int f(z)\frac{p(z)}{q(z)}q(z) dz $$
$$ \simeq \frac{Z_q}{Z_p} \frac{1}{L} \sum_{l=1}^L \tilde{r_l}f(z^{(l)}) $$

with $$\tilde{r_l} = \frac{\tilde{p}(z^{(l)})}{\tilde{q}(z^{(l)})} $$

As you might have noticed, we introduced two constants expressing our uncertaintity in the quality of our sample. These constants are not known, but we only need to know their ratio anyways. This can be done

$$ \frac{Z_q}{Z_p} = \frac{1}{Z_q}\int \tilde{p}(z)dz = \int \frac{\tilde{p}(z^{(l)})}{\tilde{q}(z^{(l)})} q(z)dz \simeq \frac{1}{L} \sum_{l=1}^L \tilde{r_l} $$

and with this we can express the expectancy as

$$ \mathbb{E}[f] \simeq \sum_{l=1}^L w_l f(z^{(l)})

with

$$ w_l = \frac{\tilde{r_l}}{\sum_m{\tilde{r_m}}} = \frac{\frac{\tilde{p}(z^{(l)})}{\tilde{q}(z^{(l)})}}{\sum_m \frac{\tilde{p}(z^{(m)})}{\tilde{q}(z^{(m)})}} $$

With this the first difference to rejection sampling should become obvious. In rejection sampling we were worried about a bad fit and throwing away to many samples. Here we use every sample and give it a weight. In all fairness though, we might keep the sample, but it still could end up getting a weight very close to zero.

