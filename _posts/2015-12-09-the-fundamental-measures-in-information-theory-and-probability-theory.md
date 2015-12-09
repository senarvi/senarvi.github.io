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
maximized. The uncertainty \(entropy\) of the random process is 1, measured in 
bits. This is also the expected amount of information an outcome of the random 
process carries. If the coin is weighted and comes up heads 100 % of the time, 
the uncertainty of the random process is 0. In that case the outcome of the 
random process gives no information. If the coin is biased, the uncertainty of 
the random process is something from between these two extremes.

Formally, the entropy <span>$H$</span> of a discrete random variable 
<span>$X$</span> is defined as

<div>$$
H(X) = -\sum_{x \in X} p(x) log_2(p(x)),
$$</div>

where <span>$x$</span> is iterated over all the possible values of 
<span>$X$</span>. In the above formula, entropy is measured with respect to base 
2. Sometimes other bases are used—then the result will not appear in bits, but 
otherwise it makes no difference in the following discussion.

If we're able to gain some information about a random process through another 
random variable, the uncertainty of the random process is diminished. 
**Conditional entropy** measures how much uncertainty in the random process 
remains, if the outcome of another random process is known. The entropy of a 
discrete random variable <span>$X$</span> conditional on the variable 
<span>$Y$</span> is

<div>$$
H(X|Y) = \sum_{y \in Y} p(y) H(X|Y=y) = -\sum_{y \in Y} \sum_{x \in X} p(x,y) log_2(\frac{p(x,y)}{p(y)}).
$$</div>

The latter form of the equation can be derived if it is understood that
<span>$0 log(0)$</span> should be treated as being equal to zero.

**Mutual information** is closely related to conditional entropy. As explained 
above, conditional entropy measures the amount of uncertainty that still remains 
about a random process <span>$X$</span>, when we know the value of another 
random variable <span>$Y$</span>. The amount the uncertainty of <span>$X$</span> 
was reduced by knowing the value of <span>$Y$</span> we call the mutual 
information of <span>$X$</span> and <span>$Y$</span>. Thus we can write

<div>$$
I(X;Y) = H(X) - H(X|Y).
$$</div>

<span>$I(X;Y)$</span>, the mutual information of <span>$X$</span> and 
<span>$Y$</span>, is a symmetric function:

<div>$$
I(X;Y) = H(Y) - H(Y|X)
$$</div>

For discrete random variables it can be written

<div>$$
I(X;Y) = \sum_{y \in Y} \sum_{x \in X} p(x,y) log_2(\frac{p(x,y)}{p(x)p(y)}).
$$</div>

If we model <span>$X$</span> with an approximate distribution 
<span>$q(x)$</span>, **cross-entropy** tells us how well <span>$q(x)$</span> 
fits the real probability distribution of X, and is defined

<div>$$
H(X,q) = -\sum_{x \in X} p(x) log_2(q(x)).
$$</div>
