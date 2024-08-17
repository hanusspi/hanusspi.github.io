---
layout: post
title: "Generating Samples from a Probability Distribution "
subtitle: "Quick Tutorial for setup and deployment of a personal free blog."
background: '/img/posts/2023-08-13-Introduction-to-ANS/header.jpg'
---
# Introduction


# Streaming rANS
  

#  Some prerequisites
Here we should see some code

```javascript
function sayHello(name) {
  if (!name) {
    console.log('Hello World');
  } else {
    console.log(`Hello ${name}`);
  }
}
```

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

