---
layout: post
title: "Approximate Inference"
subtitle: "Introduction"
background: '/img/posts/2023-08-13-Introduction-to-ANS/header.jpg'
---
# Introduction
In this little series i want to explore approximate inference a bit. To understand the purpose of approxiat inference, a quick detour to exact (bayesian) inference is needed. Maybe in the future I will give this some more attention too.


# Streaming rANS
The basic idea of streaming rANS is very simple. We define an interval $$I$$ in which the state can exist. If the state leaves this interval we read or write the least significant bit/bits to a bitstream. Now we transmit the bitstream and a final state. So far, so easy.   

#  Some prerequisites