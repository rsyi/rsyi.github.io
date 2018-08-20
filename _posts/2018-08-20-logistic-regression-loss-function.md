---
layout: post
title: A brief comment on the loss function for logistic regression
tags: machine-learning, logistic-regression, generalized-linear-models, log-loss, classification
---

Recently, the similarity of logistic to linear regression caused me to erroneously suggest that the analytical solution to logistic regression could be solved through matrix inversion (i.e. the \\(\beta = (X^T X)^{-1} X^T y\\) solution of linear regression). But this is not the case!

Note that if we were to directly follow the linear regression prescription, we would have to define a loss between \\(\boldsymbol{\beta \cdot x}\\) and the quantity that we are trying to regress. In this case, that would mean

$$\text{RSS} = \sum_{i=0}^{K-1}  \left( \log(\frac{p_k}{p_K}) - \boldsymbol{\beta \cdot x} \right)^2 $$

where \\(p_k = P(G = k \vert \boldsymbol{x})\\). Since we do not have the true value of any \\(p_k\\), but simply what is observed, the above quantity makes no sense. Or, if we took the observed value, this quantity would be \pm \infty. Instead, we want a metric that penalizes an incorrect prediction and values a correct one. What is traditionally used is the log loss function:

$$l = - \sum_{i=1}^{N} y_i \log p_i $$

Ultimately, minimizing loss function allows us to get what we desire — values that minimize loss as they approach true probabilities. However, matrix differentiation does not lead to as clean of a solution as with RSS. Consequently, we must resort to optimization schemes.
