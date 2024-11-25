---
layout: post
title:  "group codes intro"
date:   2024-11-23
tags:
- coding
---
<!--&emsp;&emsp;im writing this after bombing by second algebra midterm, so idk if its out of spite or shame or guilt or whatever other feeling. regardless, i would love to pursue research and/or a career in something related to coding theory, so this is my intro to it, specifically linear block codes. the notes are from *Abstract Algebra: Theory and Applications* by Judson.

## example: repetition code
&emsp;&emsp;suppose i am on the moon and want to send a message to my friend on mars.  due to a variety of different effects (atmospheric, cosmic, thermal, etc.), when my friend receives the message, it could be corrupted. since the message is really really important, i want to come up with a way to ensure they receive it error-free. \
&emsp;&emsp;one possible way is to simply repeat each bit in the message $k$ times. for ex:

- i repeat each bit $k = 3$ times, so i can send $000$ for each $0$ and $111$ for each $1$.
- i want my message to be $10$, so i send $111 \hspace{2pt} 000$ to my friend.
- my friend happens to receive $101 \hspace{2pt} 000$. 
    - the first three bits have two $1$ and one $0$, so by majority-rules they assume the first bit in the message is $1$.
    - the second three bits are all $0$, so they assume the second bit in the message is $0$.
- this way, they were able to *detect and correct* a single error every $3$ bits that i sent.

## the communication system

$$
\xrightarrow{m\text{-bit message}} \fbox{encoder $E$} \xrightarrow{n\text{-bit codeword}} \fbox{channel} \xrightarrow{n\text{-bit codeword + noise}} \fbox{decoder $D$} \xrightarrow{m\text{-bit received message}}
$$

&emsp;&emsp;the general comm system works as follows:

- the transmitter takes the $m$-bit message and maps it to a $n$-bit *codeword* using an encoding function $E$. the collection of all possible codewords is called the *codebook*.
- the transmitter then sends the codeword over the channel (a wire, the air, etc.), and the channel is assumed to add noise to the codeword.
- the receiver receives the noisy codeword and decodes it using decoding function $D$ to recover the original message.

&emsp;&emsp;the *rate* of a code is $m/n$. in our repetition code example, $m = 2$, $n = km = 6$, so the rate is $1/3$; for every $1$ bit of message data, we send $3$ bits over the channel. so the question is how do we maximize the rate and also make the encoding-decoding process fast?

## binary symmetric channel

&emsp;&emsp;in a binary symmetric channel, the transmitter sends a bit $0$ or $1$ with probability $p$ that the bit will not flip, and probability $q = 1 - p$ that it will. we can use the binomial pmf to calculate the probabilites of the number of errors in a message sent over this channel:

$$
\Pr{(k \text{ errors in } n\text{-bit codeword})} = \binom{n}{k}p^{n - k}q^k.
$$

&emsp;&emsp;let's do some example calculations with $p = 0.9$, i.e. there is a $10\%$ that any given bit will flip, and $n = 10$.

1) what is the probability of $0$ errors?

$$ 
\Pr{(0 \text{ errors in } 10\text{-bit codeword})} = \binom{10}{0}(0.9)^{10 - 0}(0.1)^0 = (0.9)^{10} \approx 0.349.
$$

2) what is the probability of $1$ or more errors?

$$
\Pr{(\geq 1 \text{ errors in } 10\text{-bit codeword})} = 1 - \Pr{(0 \text{ errors in } 10\text{-bit codeword})} \approx 1 - 0.349 = 0.651.
$$

## block code
&emsp;&emsp; now we will move up a gear. a $(n, m)$-block code takes a $m$-bit message, or block, and encodes it to an $n$-bit codeword. the code consists of an encoding function

$$
E : \mathbb{Z}_2^m \rightarrow \mathbb{Z}_2^n
$$

and a decoding function

$$
D : \mathbb{Z}_2^n \rightarrow \mathbb{Z}_2^m.
$$

a codeword is any element in the range, or image, of $E$. $E$ must be one-to-one so that every $m$-bit block is mapped to a unique codeword. to correct errors, $D$ must be onto. \
&emsp;&emsp;an example of a simple block code is one we can use for ascii chars. we need $7$ bits to represent all $2^7$ ascii chars, so let's make use of an eighth bit, since computers usually store information in $8$-bit multiples. the *parity* of a string of bits is even or $0$ if there are an even number of $1$, and is odd or $1$ if there is an odd number of $1$. we can compute the parity using $+$ in $\mathbb{Z}_n$, which is analogous to bitwise XOR.

- let's construct a $(8, 7)$-block code for ascii chars $x_7x_6 \dots x_1$. let $x_8 = x_7 + x_6 + \cdots + x_1$ (computing the parity of all $7$ bits of the char), which gives us an encoding function

$$
E(x_7 x_6 \dots x_1) = x_8 x_7 \dots x_1.
$$

- the encoding results in the codeword guaranteed to have even parity:
    - if the parity of the char is even, then $x_8 = 0$, and so the codeword parity is also even.
    - if the parity of the char is odd, then $x_8 = 1$, and so we make the codeword have even parity.
- when we decode, we compute the parity of the codeword. if the codeword's parity is now odd, we have successfully detected a single error.
- we are unable to detect multiple errors though, and have no way of identifying which bit has an error.

## Hamming distance
&emsp;&emsp; if we have two $m$-bit strings $\mathbf{x}, \mathbf{y}$, then the Hamming distance $d(\mathbf{x}, \mathbf{y})$ is the number of bits in which $\mathbf{x}$ and $\mathbf{y}$ differ. the distance between two codewords is the minimum number of errors required to change one codeword into the other. the minimum distance for a code, $d_{\text{min}}$ is the minimum of all distances $d(\mathbf{x}, \mathbf{y})$ where $\mathbf{x}$ and $\mathbf{y}$ are distinct codewords. these definitions allow us to state an important result about the limits of codes:

&emsp;&emsp; *let $C$ be a code with $d_{min} = 2n + 1$. then $C$ can correct any $n$ or fewer errors and any $2n$ or fewer errors can be detected.*-->