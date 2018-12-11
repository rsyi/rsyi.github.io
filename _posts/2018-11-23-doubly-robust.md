---
layout: post
title: Doubly robust estimators
tags: causal-inference
---

The goal of causal inference is to parse the effect that can be attributed to a particular treatment. In the wonderful case where we have a randomized trial, for any group of individuals, we can estimate the average treatment by simply looking at the difference between the treatment and control groups. But in the case where we don't have a randomized experiment, the problem is a lot trickier. If we want an unbiased estimator of our outcome, we typically have to either attempt to create a model that accounts for confoundedness or create a model to predict the probability of treatment. Doubly robust estimation incorporates both so that as long as we get one of them right, we end up with an unbiased estimator.

Much of the following comes from [Funk et al. (2011)](https://academic.oup.com/aje/article/173/7/761/103691) and a very nice [U.Penn coursera lecture](https://www.coursera.org/lecture/crash-course-in-causality/doubly-robust-estimators-hZjgB) on the subject.

## IPTW estimator

Inverse Probability of Treatment Weighting (IPTW) is the simple act of weighting observations by their probability of treatment $\pi$ in order to get an unbiased estimate of some outcome $Y$.

$$E_{\text{IPTW}}(Y) = \frac{\sum_i^m Y_i / \pi_i}{\sum_i^m \pi_i}$$

where $i$ sums over all observations $m$. The literature typically favors using, instead of the number of observations, the total number of individuals (observed and unobserved) $n$ and an indicator function $A$ that denotes whether or not an individual was treated or not.

$$E_{\text{IPTW}}(Y) = \frac{1}{n} \sum_i^n \frac{A_i Y_i}{\pi_i} $$

Note that both of these definitions are equivalent. **For the estimator to be unbiased, $\pi$ must be correctly specified.**

## Regression-based estimator

A regression-based estimator can be defined as follows:

$$E_{R}(Y) = \frac{1}{n} \sum \left( A_i Y_i + (1-A_i) M(X_i)\right)$$

where $M$ is a model $M(X) = E(Y\|A=1, X)$.

**For the estimator to be unbiased, $M$ must be correctly specified.**

## Doubly robust estimator

Can we combine both of these to obtain an estimator such that, as long as either is correct, we have an unbiased estimator? Consider the following quantity:

$$E_{DR}(Y) = \frac{1}{n} \sum_i^n \left( \frac{A_i Y_i}{\pi_i} - \frac{A_i - \pi_i}{\pi_i} M(X_i) \right) $$

Imagine what would happen if we have a good propensity score model $\pi$, but our model $M$ is incorrect. In this case, the first term remains unaffected. What happens to the second term? We are essentially asking what is the expected value of the second term?

$$E\left( \frac{A_i - \pi_i}{\pi_i} M(X_i) \right)\\
= E(A_i M(X_i)/\pi_i) - E(M(X_i)) $$

We are weighting each $M(X_i)$ by the inverse probability of treatment, so $E(A_i M(X_i) / \pi_i) = E(M(X_i))$. This quantity, therefore, goes to 0.

Similarly, we can rearrange the doubly robust estimator as follows and ask what happens when our propensity score model $\pi$ is not correct, but $M(X_i)$ is:

$$E_{DR}(Y) = \frac{1}{n} \sum_i^n \left( \frac{A_i(Y_i - M(X_i))}{\pi_i} + M(X_i) \right) $$

If our model is perfect, $Y_i = M(X_i)$, and so the first term goes to $0$, leaving us with $E(M(X_i))$. Therefore, as long as we get either our model of the treatment $\pi$ or our model of the outcome $M$ correct, our estimator will be unbiased.
