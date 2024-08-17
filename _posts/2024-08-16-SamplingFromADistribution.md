---
layout: post
title: "Generating Samples from a Probability Distribution "
subtitle: "Quick Tutorial for setup and deployment of a personal free blog."
background: '/img/posts/2023-08-13-Introduction-to-ANS/header.jpg'
---
# 1. What is Bayesian Inference?

Bayesian inference is a powerful statistical technique that allows us to update our beliefs about the world as we gather more evidence. Unlike classical (frequentist) approaches, which often focus on fixed parameters and p-values, Bayesian inference provides a probabilistic framework where uncertainty is quantified and incorporated into the analysis.

At its core, Bayesian inference is built upon Bayes' Theorem:

$$P(H|D) = \frac{P(D|H)\cdot P(H)}{P(D)}$$

* a $$P(H|D)$$: The posterior probability, which represents our updated belief in the hypothesis H after observing data D.
* a $$P(D|H)$$: The likelihood, the probability of the data given that the hypothesis H is true.
* a $$P(H)$$: The prior probability, which represents our initial belief in the hypothesis before seeing the data.
* a $$P(D)$$: The evidence or marginal likelihood, which is the total probability of observing the data under all possible hypotheses.

Bayesian inference is particularly useful when dealing with uncertainty, incorporating prior knowledge, and updating beliefs as new information becomes available.

# 2. Exact vs. Approximate Inference
  
While Bayesian inference provides a robust framework, it can be computationally challenging. In simple cases, we can calculate the posterior distribution exactly, which is known as exact Bayesian inference. However, in more complex models, calculating the exact posterior distribution becomes intractable due to high-dimensional integrals or complex likelihood functions.

This is where approximate inference comes into play. Approximate methods aim to estimate the posterior distribution without solving it exactly. These methods allow us to perform Bayesian inference in situations where exact methods would be computationally impossible.

Here’s a brief comparison:

   * Exact Inference:
       - Pros: Provides precise posterior distributions.
       - Cons: Computationally expensive or impossible for complex models.

   * Approximate Inference:
       - Pros: Scalable to complex models and large datasets.
       - Cons: Introduces approximations, which may lead to less precise results.

This blog series will dive into several approximate methods, but before we do, let’s walk through a simple example of exact Bayesian inference.

#  3. Example of Exact Bayesian Inference
Let’s consider a classic problem in Bayesian statistics: estimating the probability of a coin being biased.

Imagine you have a coin, and you're not sure whether it’s fair (i.e., whether it lands heads or tails with equal probability). You decide to model this uncertainty using Bayesian inference.

* Prior Belief: You believe that the coin is probably fair, but you're open to the possibility that it might be biased. This belief can be represented by a Beta distribution, a common choice for modeling probabilities.

* Data: You flip the coin 10 times and observe 7 heads.

* Likelihood: The likelihood function here is binomial since we're counting the number of successes (heads) in a fixed number of trials (10 flips).

* Posterior Distribution: Using Bayes' Theorem, we update our belief about the fairness of the coin.

Here’s the Python code that performs exact Bayesian inference for this problem:

{% highlight python linenos %}
import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import beta

# Define prior: Beta(2, 2) (initial belief that the coin is fair)
a_prior = 2
b_prior = 2

# Data: 7 heads out of 10 flips
heads = 7
tails = 3

# Posterior distribution parameters: Beta(a + heads, b + tails)
a_post = a_prior + heads
b_post = b_prior + tails

# Generate x values for plotting
x = np.linspace(0, 1, 100)
{% endhighlight %}

