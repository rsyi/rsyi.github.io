---
layout: post
title: The assumption of linearity behind logistic regression
tags: machine-learning generalized-linear-models
---

Logistic regression is an adaptation of linear regression used to predict categorical outcomes. But the linearity is not directly apparent, and generally left undiscussed. In using logistic regression, we are actually assuming that **the ratio between any two probabilities scales linearly with the features**. I’ll briefly lay out why this is the case.

In the two-class case (for simplicity), the predicted value of an outcome \\(G=1\\) occurring is assumed sigmoidal:

$$P(G = 1 \vert \boldsymbol{x}) = \frac{\exp(\boldsymbol{\beta \cdot x})}{1 + \exp(\boldsymbol{\beta \cdot x})} \ \ \ (1)$$

Consequently, it can be easily shown that the predicted value of the second class \\(G=0\\) occurring is also a sigmoid. The resulting values sum to 1 and fall between 0 and 1. Therefore, given the correct loss function, they can be interpreted as probabilities.

To determine what $$\boldsymbol{\beta \cdot x}$$ is regressing, invert (1). The sigmoid, \\(y = \frac{1}{1 + \exp(-x)}\\) is inverted by the logit function, \\(x = \log(\frac{y}{1-y})\\). The logit (or log-odds) of \\(p = P(G=k \vert \boldsymbol{x})\\), then, can be written as follows:

$$\begin{aligned} \text{logit}(p) &= \log\left(\frac{p}{1-p}\right) \\ &= \boldsymbol{ \beta \cdot x }. \end{aligned}$$

We are therefore assuming that *the log-odds are linear in our features \\(\boldsymbol{x}\\)*.

To generalize this, I’ll borrow the multinomial logistic regression definition from Elements of Statistical Learning, in which \\(P(G = k \vert \boldsymbol{x})\\) is defined as:

$$P(G=k \vert \boldsymbol{x}) = \frac{\exp(\boldsymbol{\beta_k \cdot x})}{1 +\sum_{i=1}^{K-1} \exp(\boldsymbol{\beta_i \cdot x}) }$$

$$P(G= K \vert \boldsymbol{x} ) = \frac{1}{1 +\sum_i^{K-1} \exp(\boldsymbol{\beta \cdot x}) }$$

In this case, we do not assume the log-odds are linear, but that *the log of the ratio between G=k and a reference class G=K is linear in \\(\boldsymbol{x}\\)*.

Note that this formulation is actually equivalent to softmax regression, in which

$$P(G=k \vert \boldsymbol{x}) = \frac{\exp(\boldsymbol{\beta_k \cdot x})}{\sum_{i=1}^K \exp(\boldsymbol{\beta_i \cdot x})},$$

but with different coefficients \\(\beta_k\\). For two classes \\(g_1\\) and \\(g_2\\), we can thus state our assumption more generally:

$$p_{g_1}/p_{g_2} \sim \boldsymbol{x}. \ \ \ (2)$$

<div style="padding: 10px; background-color: #ccc; font-size: 1.3em;">
Therefore, logistic regression assumes that the ratios between probabilities of different categorical outcomes scale linearly with $\boldsymbol{x}$. Any feature engineering then, should reflect your belief about how these are related.
</div>
