---
layout: post
title: The assumption behind logistic regression
---

Logistic regression is an adaptation of linear regression used to predict categorical outcomes. But the linearity is not directly apparent, and generally left undiscussed. In using logistic regression, we are actually assuming that the ratio between any two probabilities scales linearly with the features. I’ll briefly lay out why this is the case.

In the two-class case (for simplicity), the predicted value of an outcome \\(G=1\\) occurring is assumed sigmoidal:

$$P(G = 1 | \boldsymbol{x}) = \frac{\exp(\boldsymbol{\beta \cdot x})}{1 + \exp(\boldsymbol{\beta \cdot x})} \ \ \ (1)$$

