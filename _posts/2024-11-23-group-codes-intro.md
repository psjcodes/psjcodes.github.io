---
layout: post
title:  "group codes intro"
date:   2024-11-23
tags:
- coding
---
&emsp;&emsp;im writing this after bombing by second algebra midterm, so idk if its out of spite or shame or guilt or whatever other feeling. regardless, i would love to pursue research and/or a career in something related to coding theory, so this is my intro to it. the notes are from *Abstract Algebra: Theory and Applications* by Judson.

## detecting and correcting error
&emsp;&emsp;suppose i am on the moon and want to send a message to my friend on mars. the message is in the form of a binary $m$-tuple $\(x_1, x_2, \dots, x_n\)$. due to a variety of different effects (atmospheric, cosmic, thermal, etc.), when my friend receives the message, it could be corrupted. since the message is really really important, i want to come up with a way to ensure they receive it error-free. \
&emsp;&emsp;one possible way is to simply repeat each 
- my message is $(101)$, so i send $(111 \hspace{2pt} 000 \hspace{2pt} 111)$
- my friend receives $(101 \hspace{2pt} 000 \hspace{2pt} 111)$
- w
- this way, we were able to *detect and correct* a single error 
