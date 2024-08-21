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

For the weights we can see, that the sum acts as a normalization factor. With this the first difference to rejection sampling should become obvious. In rejection sampling we were worried about a bad fit and throwing away to many samples. Here we use every sample and give it a weight. In all fairness though, we might keep the sample, but it still could end up getting a weight very close to zero.

Now lets take a look about the resulting implications.
* The success of importance sampling depends still crucially on how well the sampling distribution $$q(z)$$ matches the desired target distribution $$p(z)$$
* Often, $$ p(z)f(z) $$ is strongly varying and has a significant proportion of its mass concentrated over small regions of the z-space
* Expectancy may be dominated by a few heavy weights
* The results can be wrong and produce arbitraty errors with no diagnostic indications. This will happen, if the proposal distribution is to far off or the samples are by random chance in none of the relevant areas of $$ p(z)f(z) $$

# 3. Example of Importance Sampling

Lets now take a look at how this can look in code:

{% highlight python linenos %}
import numpy as np
from scipy.integrate import quad
import matplotlib.pyplot as plt
from scipy.stats import norm

np.random.seed(42)

# Define the lower and upper bounds of the integral
lower_bound = -2
upper_bound = 8
# Define the target distribution (Mixture of Gaussians)
def target_distribution(x):
    return 0.3 * norm.pdf(x, 1, 0.9) + 0.7 * norm.pdf(x, 5, .6)

# Define the proposal distribution (Gaussian distribution)
def proposal_distribution(x):
    return norm.pdf(x, 3, 2)

# Define the importance sampling function
def importance_sampling(num_samples):
    samples = np.random.uniform(lower_bound, upper_bound, num_samples)  # Generate samples from the proposal distribution
    weights = target_distribution(samples) / proposal_distribution(samples)  # Compute the importance weights
    weights /= np.sum(weights)  # Normalize the weights
    
    # Estimate the expectation of a function using importance sampling
    def estimate_expectation(func):
        return np.sum(func(samples) * weights)

    return estimate_expectation

def combined(x):
    return target_distribution(x)*sigmoid(x)

# Define the exact solution
def exact_solution():
    return quad(combined, lower_bound, upper_bound)[0]

# Define the sigmoid function
def sigmoid(x):
    return 1 / (1 + np.exp(-x+3))-.2

# Perform importance sampling with 10000 samples
estimate = importance_sampling(100)(sigmoid)
exact = exact_solution()

print('Estimate:', estimate)
print('Exact:', exact)

# Print the estimates
x = np.linspace(lower_bound, upper_bound, 1000)  # Define "x" as an array of values
plt.plot(x, sigmoid(x), label='Sigmoid')
plt.plot(x, proposal_distribution(x), label='Proposal Distribution')
plt.plot(x, target_distribution(x), label='Target Distribution')

plt.axhline(y=0, color='black', linestyle='--')
plt.legend()
plt.show()
{% endhighlight %}
![Imagetext](img\posts\2024-08-19-Importance-Sampling.md\importance_sampling.jpg)

So, what is happening here. First we define our target function, which is a Mixture of Gaussians, then our proposal distribution, which is a simple normal distribution and $$f(x)$$ is a sigmoid function. Then we generate the samples and calculate the corresponding weights and their normalization factor and get the following expectancy for the importance sampled expectancy: 0.413 and using numerical integration we get an "exact" value of:  0.451, which seems to be a pretty good guess. But moving the mean of our propsal distribution by 2 to the left turns our estimate already to 0.705 and keeping the mean at 3 and changing the variance to 1 turns the expactancy to -0.05. If we visulize the functions, we can also see why:
![Imagetext](img\posts\2024-08-19-Importance-Sampling.md\importance_sampling_bad.jpg)
The proposal distribution only covers the low part of the target distribution leading to the bad result. 

# 4. Conclusion and What's Next

In this post, we've introduced a more versatile sampling method than rejection sampling. We implemented rejection sampling in Python and used the example to better understand the implications of importance sampling. 

In the next post, we'll explore a more universal approach using the Metropolis Hastings Algorithm with Markov Chains 





