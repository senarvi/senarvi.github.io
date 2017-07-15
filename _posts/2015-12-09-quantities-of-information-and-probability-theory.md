---
layout: post
title: "Quantities of information and probability theory"
description: "Entropy, conditional entropy, mutual information, cross entropy, Kullback-Leibler divergence, and perplexity"
category: 
tags: []
use_math: true
---
{% include JB/setup %}

### Entropy

In mathematical statistics, **entropy** measures the uncertainty in the outcome
of a random phenomenon. Consider the random experiment of tossing a coin. If the
coin is fair, entropy is maximized. The uncertainty \(entropy\) in the
experiment is 1, measured in bits. This is also the expected amount of
information an outcome of the random experiment carries. If the coin is weighted
and comes up heads 100 % of the time, the uncertainty of the random experiment
is 0. In that case the outcome of the random experiment gives no information. If
the coin is biased, the uncertainty of the random experiment is something from
between these two extremes.

Formally, the entropy <span>$H$</span> of a discrete random variable
<span>$X$</span> is defined as

<div>$$
H(X) = -\sum_{x \in X} p(x) \log_2(p(x)),
$$</div>

where <span>$x$</span> is iterated over all the possible values of
<span>$X$</span>. In the above formula, entropy is measured with respect to base
2\. Sometimes other bases are used—then the result will not appear in bits, but
otherwise it makes no difference in the following discussion.

### Conditional entropy and mutual information

If we're able to gain some information about a random phenomenon through another
random phenomenon, our uncertainty is diminished. **Conditional entropy**
measures how much uncertainty in the outcome of one random phenomenon remains,
if the outcome of another random phenomenon is known. The entropy of a discrete
random variable <span>$X$</span> conditional on the variable <span>$Y$</span> is

<div>$$
H(X \mid Y) = \sum_{y \in Y} p(y) H(X \mid Y=y)
            = -\sum_{y \in Y} \sum_{x \in X} p(x,y) \log_2(\frac{p(x,y)}{p(y)}).
$$</div>

The latter form of the equation can be derived if it is understood that
<span>$0 \log(0)$</span> should be treated as being equal to zero.

**Mutual information** is closely related to conditional entropy. As explained 
above, conditional entropy measures the amount of uncertainty that still remains 
about a random variable <span>$X$</span>, when we know the value of another 
random variable <span>$Y$</span>. The amount the uncertainty of <span>$X$</span> 
was reduced by knowing the value of <span>$Y$</span> we call the mutual 
information of <span>$X$</span> and <span>$Y$</span>. Thus we can write

<div>$$
I(X;Y) = H(X) - H(X \mid Y).
$$</div>

&#x20;<span>$I(X;Y)$</span>, the mutual information of <span>$X$</span> and 
<span>$Y$</span>, is a symmetric function:

<div>$$
I(X;Y) = H(Y) - H(Y \mid X)
$$</div>

For discrete random variables it can be written

<div>$$
I(X;Y) = \sum_{y \in Y} \sum_{x \in X} p(x,y) \log_2(\frac{p(x,y)}{p(x)p(y)}).
$$</div>

### Cross entropy and Kullback-Leibler divergence

If we model <span>$X$</span> with an approximate distribution 
<span>$q(x)$</span>, **cross entropy** tells us how well <span>$q(x)$</span> 
fits the real probability distribution of <span>$X$</span>, and is defined

<div>$$
H(X,q) = -\sum_{x \in X} p(x) \log_2(q(x)).
$$</div>

When <span>$p(x) = q(x)$</span>, cross entropy has its minimum value, which is 
the entropy <span>$H(p)$</span>. Another similar measure is the **Kullback-Leibler 
divergence** or **relative entropy** (also sometimes called cross entropy):

<div>$$
D_{KL}(p \mid q) = H(X,q) - H(X)
                 = \sum_{x \in X} p(x) \log_2(\frac{p(x)}{q(x)})
$$</div>

Kullback-Leibler divergence is non-negative and zero if and only if
<span>$p(x) = q(x)$</span> almost everywhere. Intuitively Kullback-Leibler
divergence can be seen as a difference between two distributions, but it is not
a true metric and not a symmetric function. Specifically, it measures the
information lost, when <span>$q(x)$</span> is used to approximate
<span>$p(x)$</span>.

### Perplexity

Perplexity is a measure for the uncertainty of a random phenomenon, closely
related to entropy and cross entropy. The perplexity of a random variable
<span>$X$</span> is defined either

<div>$$
2^{H(X)} = 2^{-\sum_{x \in X} p(x) \log_2(p(x))},
$$</div>

or

<div>$$
2^{H(X,q)} = 2^{-\sum_{x \in X} p(x) \log_2(q(x))}.
$$</div>

We can easily see that the perplexity of a fair coin is 2 (we are “two ways 
perplexed” about the outcome of the random experiment).

### Information content

Another perspective to these measures comes from information theory, where the 
possible values of <span>$X$</span> are seen as messages to be coded using an 
optimal coding scheme. Entropy, then, gives the expected message length, 
cross entropy gives the expected message length when the coding scheme is 
optimal for <span>$q(x)$</span> but the data follows <span>$p(x)$</span>, and KL 
divergence gives the extra message length when the coding scheme is optimal for 
<span>$q(x)$</span> but the data follows <span>$p(x)$</span>.

Confusingly, entropy is sometimes used to refer to the quantity of information
that the outcome of a random variable contains. This quantity is also called the
**self-information**, and defined as:

<div>$$
I(X=x) = -\log_2(p(x))
$$</div>

Entropy is the expected value of self-information.

For example, when tossing a fair coin, the probability that the outcome is heads
is 0.5, so the self-information of that event is <span>$-\log_2(0.5) = 1$</span>.
The event “heads” is said to carry 1 bit of information.

### Estimation from a sample and language modeling

Often we would like to measure these quantities, but instead of the true
probability distribution we have a sample drawn from it. There are actually many
ways to estimate the entropy from a sample; one possibility is to estimate each
of the probabilities <span>$p(x)$</span> as the fraction of <span>$x$<span> in
the sample. To compare a known distribution <span>$p(x)$</span> to a sample
<span>$x_1 x_2 \ldots x_N$</span>, the Monte Carlo estimate of cross entropy can
be computed as:

<div>$$
H(x_1 \ldots x_N,p) = -\frac{1}{N} \sum_i \log_2(p(x_i))
$$</div>

For example, in natural language processing, it is usual to model a sequence of
words <span>$w_1 w_2 \ldots w_N$</span> as the outcome of a random phenomenon
whose probability is given by a language model:

<div>$$
p(w_1 \ldots w_N) = \prod_i p(w_i \mid w_1 \ldots w_{i-1})
$$</div>

Cross entropy can be used to evaluate how well the model predicts a given text
sequence. In this context, cross entropy is defined as if the words were
independent observations:

<div>$$
H(w_1 \ldots w_N,p) = -\frac{1}{N} \sum_i \log_2(p(x_i)) = -\frac{1}{N} \log_2(p(w_1 \ldots w_N))
$$</div>

and perplexity is defined as the exponent of cross entropy:

<div>$$
PP(w_1 \ldots w_N) = 2^{H(w_1 \ldots w_N,p)}
                   = 2^{-\frac{1}{N} \log_2(p(w_1 \ldots w_N))}
                   = \frac{1}{p(w_1 \ldots w_N)}^\frac{1}{N}
$$</div>

The latter form of the equation brings us another, equal, definition for the 
perplexity of a word sequence as the geometric mean of the word conditional 
probabilities:

<div>$$
PP(w_1 \ldots w_N) = \sqrt[N]{\frac{1}{p(w_1) p(w_2 \mid w_1) \ldots p(w_N \mid w_1 \ldots w_{N-1})}}
$$</div>
