---
layout: post
title: "Introduction to ANS"
subtitle: "because they lacked opposable thumbs and the brainpower to build a space program."
background: '/img/posts/01.jpg'
---
# Work in Progress
I came in contact with Asymmetric Numeral Systems first one year ago, when i looked deeper into entropy coding. Whilst stumbling over huffman coders and arithmetic coding, i crossed roads with ANS. At first glance it is surprising, how with those two old coders, which basicly solved entropy coding, such a new development took place. Delving deeper into all those coders, it becomes easy to notice the shortcomings of the two old coders and the elegance of ANS. The industry supports this view, by including ANS  in Zstandard, LZFSE, CRAM and many more. Unterstanding ANS at its full took me quite a while (and propably i am still not done with the process). So let me take you along the journey i took and hopefully make unterstanding ANS a little easier.

#  Some prerequisites
For everyone, who isnt that familar the fundamentals of information theory and entropy coding, lets talk some fundamentals. The core concept of information theory is to send as much information through a bandwith limited chanel as possible. But what is information? Information is a combination of any number of different symbols with meaning to two parties. The sender and the reciever. Now to quantify information, we need to make some definitions. We call the sender source and the source emits symbols $$s$$ from an Alphabet $$A$$ with properbilites for each symbol $$p_s$$. All those properbilites add to one. To keep it simple, we are going to assume, that the probabilites are independent from any previously emitted symbols. Some short dependencies can be fitted in this pattern, by defining a sequence as a unique symbol.

Seeing that each symbol needs to have some sort of informative value, it can be assumed, that this value soly depends on the propability, with which it is emitted. Now we say, that the more frequently a symbol it transimitted, the less suprising it becomes to the reciever, so its informational value is less then a symbol, that is seldomly emitted. Finaly we need a unit, to meassure the information and for that we take bits. Coming from this, the information of a symbol is:

$$I_s = \operatorname{ld}(p_s)$$

With that we can define, how much Information a source emitts on average with every symbol by calculating the weighted average, which is called entropy of a source:

$$H = \sum p_s * I_s$$

Knowig this, we can cleary understand the goal of entropy coding. The output of the entropy encoder should match the information output of the source. The average output of the encoder will be refered to as bitrate $$R$$. Now we can meassure the difference of the bitrate and entropy and called that coding redundancy $$\Delta R = R-H$$. So by minimizing the coding redundancy we can optimize the entropy coder. As far as the concept of information theory goes, the entropy marks the low bound for the encoder. So any further compression will lead to a loss of information.

One way, to achieve minimum coding redundacy is to give every symbol of the alphabet a codeword, aka a bitsequence, of variable length, matching the information of the symbol. Assuming all propabilities would be roughly similar, that would lead to equal length codewords, which is real easy and wont require any complex fiddling with entropy coding. So we only want to use data, with extrem symbol distributions. Giving each symbol a codeword of variable length matching its information is exactly, what huffman coding does. But by doing this, we will run into a few core issues. First, the minimum encodable information is one bit. So if we have a symbol with less information, we loose bandwith. This continues, as we are limitied to full bit information values, whereas real data doesnt follow these propabilited, again introducing a loss of bandwith and therby creating a necessity for new coders. Enter ANS

#  ANS
To make ANS really simple, we take a state $$x$$ and a symbol $$s$$ and create a state machine that turns those two into a new state $$x'$$. For decoding the new state must be created in a way, that it reliably returns the old state $$x$$ and the encoded symbol $$s$$. Further more for optimal encoding, the bitrepresentation of the new state needs to be the same size as the sum of the bit representation of the old state and the information of the symbol. One super simple example of this principle would be simulating a binary source emitting symbols with the same frequency. Now encoding means turning a binary number into a decimal and decoding doing the reverse. And this process can be done sequentially


+ Entropiekodierung
+ uABS
+ rANS
+ tANS
## Testheader
Some text

## another Test
some more text

# and some more stuff
![Imagetext](/img/posts/01.jpg)

$$ y = x^2 $$

$$ x = \frac{4}{8} $$

okay, some more text

And finally the test text