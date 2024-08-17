---
layout: post
title: "Rejection Sampling: A Powerful Technique for Sampling from Complex Distributions"
subtitle: "Understanding Rejection Sampling and Its Applications in Bayesian Inference with Python"
background: '/img/posts/2023-08-13-Introduction-to-ANS/header.jpg'
---
# 1. Introduction to Rejection Sampling

When dealing with complex probability distributions, direct sampling can be difficult or even impossible. This is where Rejection Sampling comes in—a simple yet powerful technique that allows us to sample from a complex distribution by leveraging a simpler distribution.

Rejection sampling is especially useful in Bayesian inference when dealing with non-standard distributions that don't have a straightforward method for drawing samples. In this post, we'll explore the concept of rejection sampling, how it works, and how to implement it in Python.

# 2. Why Rejection Sampling?

Rejection sampling is used when:

* Direct Sampling is Difficult: When you can't easily sample from the target distribution, rejection sampling provides an alternative method.

* Target Distribution is Known Up to a Constant: Often, the target distribution is known only up to a normalizing constant, making it hard to sample directly. Rejection sampling circumvents this by sampling from a simpler distribution.

* Flexibility: Rejection sampling can be applied to a wide range of distributions, making it a versatile tool in Bayesian inference and other areas of statistics and machine learning.

# 3. How Rejection Sampling Works

The basic idea of rejection sampling is to sample from a proposal distribution and then "reject" or "accept" samples based on how likely they are under the target distribution.

Here's a step-by-step breakdown:

1. Choose a Proposal Distribution: Select a proposal distribution $$q(x)$$ that is easy to sample from and that roughly covers the shape of the target distribution $$p(x)$$. The proposal distribution should be scaled by a constant $$M$$, such that $$Mq(x)$$ upper-bounds $$p(x)$$.

2. Generate a Sample: Draw a sample $$x$$ from the proposal distribution $$q(x)$$.

3. Generate a Uniform Random Variable: Draw a uniform random variable $$u$$ from $$Uniform(0,1)$$.

4. Acceptance Criterion: Accept the sample $$x$$ if $$u\leq\frac{p(x)}{Mq(x)}$$​. If not, reject the sample and go back to step 2.

5. Repeat: Continue the process until you have enough accepted samples.

The accepted samples will be approximately distributed according to the target distribution $$p(x)$$.

# 4. Example: Rejection Sampling from a Gaussian Mixture Model

Let's consider an example where we want to sample from a Gaussian Mixture Model (GMM), which is a combination of multiple Gaussian distributions. This is a common scenario where rejection sampling is useful because direct sampling from a GMM can be challenging.

We'll use a simple GMM composed of two Gaussian distributions.

The target distribution $$p(x)$$ is a mixture of two Gaussian distributions:
$$p(x)=0.3\cdot \mathcal{N}(x;−2,0.5^2)+0.7\cdot \mathcal{N}(x;2,1^2)$$

We will use a simple normal distribution as the proposal distribution q(x)q(x).

5. Python Code Example: Rejection Sampling

{% highlight python linenos %}
import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import norm

# Define the target distribution (Gaussian Mixture Model)
def target_distribution(x):
    return 0.3 * norm.pdf(x, -2, 0.5) + 0.7 * norm.pdf(x, 2, 1.0)

# Define the proposal distribution (Normal distribution)
proposal_distribution = norm(0, 3)  # Mean 0, std 3

# Define the constant M such that M * proposal_distribution >= target_distribution
M = 1.5  # Chosen to ensure the proposal upper-bounds the target

# Number of samples
n_samples = 10000

# Rejection Sampling
samples = []
while len(samples) < n_samples:
    x = proposal_distribution.rvs()  # Sample from proposal distribution
    u = np.random.uniform(0, 1)  # Uniform random variable

    # Accept or reject the sample
    if u <= target_distribution(x) / (M * proposal_distribution.pdf(x)):
        samples.append(x)

# Convert to numpy array for easier handling
samples = np.array(samples)

# Plot the histogram of the samples
plt.figure(figsize=(10, 6))
plt.hist(samples, bins=50, density=True, alpha=0.6, color='b', label="Sampled Data")

# Overlay the target distribution
x = np.linspace(-6, 6, 1000)
plt.plot(x, target_distribution(x), 'r', linewidth=2, label="Target Distribution")
plt.title("Rejection Sampling from a Gaussian Mixture Model")
plt.xlabel("Value")
plt.ylabel("Density")
plt.legend()
plt.show()
{% endhighlight %}

![Imagetext](/img/posts/2024-08-17-RejectionSampling/rejectionSampling.jpg)

# 5. Applications and Limitations of Rejection Sampling

Applications:

* Bayesian Inference: Rejection sampling is used to sample from posterior distributions that are difficult to sample directly.
* Monte Carlo Methods: Rejection sampling is a fundamental technique in Monte Carlo simulations, where samples from complex distributions are needed.
* Importance Sampling: Rejection sampling can be combined with importance sampling to improve efficiency in certain scenarios.

Limitations:

* Efficiency: Rejection sampling can be inefficient if the proposal distribution doesn't closely match the target distribution. The more samples you reject, the longer it takes to generate enough accepted samples.
* Dimensionality: In high-dimensional spaces, finding a suitable proposal distribution that covers the target distribution well can be challenging, leading to a high rejection rate.

# Conclusion and What's Next

In this post, we've introduced the concept of rejection sampling, a versatile technique for sampling from complex distributions. We implemented rejection sampling in Python and visualized how the method works in practice. Rejection sampling is a powerful tool but comes with trade-offs in terms of efficiency and complexity.

In the next post, we'll explore Importance Sampling, a related method that improves efficiency by weighting samples from the proposal distribution instead of rejecting them.