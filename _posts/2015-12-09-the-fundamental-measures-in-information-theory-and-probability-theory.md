---
layout: post
title: "The fundamental measures in information theory and probability theory"
description: "Entropy, mutual information, cross-entropy, Kullback-Leibler divergence, and perplexity"
category: 
tags: []
use_math: true
---
{% include JB/setup %}

**Entropy** measures the uncertainty in the outcome of a random process. 
Consider the random process of tossing a coin. If the coin is fair, entropy is 
maximized. The uncertainty \\(entropy\\) of the random process is 1, measured in 
bits. This is also the expected amount of information an outcome of the random 
process carries. If the coin is weighted and comes up heads 100 % of the time, 
the uncertainty of the random process is 0. In that case the outcome of the 
random process gives no information. If the coin is biased, the uncertainty of 
the random process is something from between these two extremes.

Formally, the entropy <span>$H$</span> of a discrete random variable 
<span>$X$</span> is defined as

<div>$$
H(X) = -\sum_{x \in X} p(x) log_2(p(x))
$$</div>

where <span>$x$</span> is iterated over all the possible values of 
<span>$X$</span>. In the above formula, entropy is measured with respect to base 
2. Sometimes other bases are used—then the result will not appear in bits, but 
otherwise it makes no difference in the following discussion.
