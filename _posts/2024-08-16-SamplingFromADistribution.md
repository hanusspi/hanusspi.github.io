---
layout: post
title: "Sampling from a Distribution "
subtitle: "Understanding Sampling and Its Importance in Bayesian Inference with Python"
background: '/img/posts/2023-08-13-Introduction-to-ANS/header.jpg'
---
# 1. Introduction to Sampling

Sampling is a foundational concept in statistics and machine learning, and it plays a crucial role in Bayesian inference, particularly in approximate methods. Simply put, sampling involves drawing random values from a probability distribution. These samples can be used to estimate properties of the distribution, such as its mean, variance, or other moments.

In this post, we’ll explore the basics of sampling, its significance in Bayesian inference, and how to perform sampling from a specific distribution using the Inverse Transform Sampling technique.
# 2. Why Sampling is Important in Inference

Sampling is essential in approximate inference methods, especially when dealing with complex models where exact analytical solutions are not feasible. It allows us to estimate expectations, probabilities, and other quantities by drawing samples from the desired distribution.

This is crucial in many applications, including:

* Monte Carlo Methods: Approximating expectations using random samples.
* Bayesian Inference: Sampling from the posterior distribution when it cannot be computed analytically.
* Optimization: Exploring complex probability landscapes using sampling techniques.

In this post, we’ll focus on a practical example of sampling from an exponential distribution using Python.
# 3. The Exponential Distribution

The exponential distribution is a common distribution used to model the time between independent events that happen at a constant average rate. It is characterized by the rate parameter $$λ$$.

The probability density function (PDF) of the exponential distribution is given by:
$$f(x;λ)=λe−λxforx≥0$$

where $$λ$$ is the rate parameter.

The cumulative distribution function (CDF) is the integral of the PDF:
$$F(x;λ)=1−e−λx$$

Using the inverse of this CDF, we can sample from the exponential distribution using a uniform random variable.

# 4. Inverse Transform Sampling

Inverse Transform Sampling is a method to generate random samples from a given distribution by inverting its CDF. The steps are:

    Generate a uniform random variable $$u∼Uniform(0,1)$$.
    Compute the inverse of the CDF at uu to obtain the corresponding sample xx.

For the exponential distribution, the CDF is:
$$F(x;λ)=1−e−λx$$

To invert this, solve for xx:
$$u=1−e−λx$$
$$e−λx=1−u$$
$$x=−1λln⁡(1−u)$$

Thus, to generate a sample from the exponential distribution, we can use the formula:
$$x=−1λln⁡(1−u)$$

# 5. Python Code Example: Sampling from an Exponential Distribution

Here’s how you can implement inverse transform sampling for the exponential distribution in Python:

{% highlight python linenos %}

import numpy as np
import matplotlib.pyplot as plt

# Define the rate parameter lambda
lambda_param = 1.0

# Number of samples
n_samples = 1000

# Generate uniform random variables
u = np.random.uniform(0, 1, n_samples)

# Inverse transform sampling to generate samples from the exponential distribution
samples = -np.log(1 - u) / lambda_param

# Plot the histogram of the samples
plt.figure(figsize=(10, 6))
plt.hist(samples, bins=30, density=True, alpha=0.6, color='b')
plt.title("Sampling from an Exponential Distribution using Inverse Transform Sampling")
plt.xlabel("Value")
plt.ylabel("Density")

# Overlay the true exponential distribution
x = np.linspace(0, 8, 1000)
plt.plot(x, lambda_param * np.exp(-lambda_param * x), 'r', linewidth=2, label='True PDF')
plt.legend()
plt.show()

{% endhighlight %}

![Imagetext](img/posts/2024-08-16-SamplingFromADistribution/samplingFromDistr.jpg)

# 6. Applications of Sampling

Sampling isn’t just a theoretical exercise; it has practical applications in various domains:

* Monte Carlo Methods: Sampling forms the basis of Monte Carlo methods, which are used to estimate integrals, simulate systems, and perform Bayesian inference.
* Bootstrap Methods: In statistics, sampling is used in bootstrap methods to estimate the sampling distribution of a statistic by resampling from the data.
* MCMC Algorithms: Markov Chain Monte Carlo methods rely heavily on sampling to explore complex posterior distributions.

These applications show how sampling is a versatile tool in both statistics and machine learning.

# Conclusion and What's Next

In this post, we’ve introduced sampling from the exponential distribution using the Inverse Transform Sampling technique. Sampling is a crucial technique in Bayesian inference, especially when exact solutions are not feasible.

In the next post, we’ll dive into Rejection Sampling, a powerful method for sampling from complex distributions when direct sampling is difficult.

