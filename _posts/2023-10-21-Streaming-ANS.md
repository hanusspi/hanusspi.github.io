---
layout: post
title: "Streaming rANS"
subtitle: "ANS, but now in useable"
background: '/img/posts/2023-08-13-Introduction-to-ANS/header.jpg'
---
# Introduction
In my last post i began a deep dive into the rabit hole of entropy coding using ANS. Now it is time to go even deeper. So far we used a single integer to represent our state. Using python this would be no problem, as there are unlimited datatypes. Using any other language (and aspects of time effiency) force us to limit the size of the state. What would be suitable approaches. The naive approach would be to just define a max state and everytime the encoder would break through that state we transmit/store the integer. This idea looks good until we remember the thoughts about optimality. In the last blog i showed some test to optimality, which showed, that ANS only really works for long messages. Those messages would require states far greater then typical usable uint datatypes. So with this naive approach we loose all of our compression. For that reason Duda suggested a far superior approach. 

# Streaming rANS
The basic idea of streaming rANS is very simple. We define an interval $$I$$ in which the state can exist. If the state leaves this interval we read or write the least significant bit/bits to a bitstream. Now we transmit the bitstream and a final state. So far, so easy.   

#  Some prerequisites





