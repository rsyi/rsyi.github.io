---
layout: post
title: Doubly robust estimators
tags: causal-inference
---

We often want to obtain an unbiased estimate of a quantity $Y$, but all we have is a biased dataset. In particular, this is serious problem in **causal inference**, where we want to parse the effect of a treatment. Without unbiased estimates of an outcome with respect to both a treated group and a control group, there is not much we can actually infer! However, this concept is generalizable to any estimate of a quantity where it's not clear if data collection was unbiased, and we want to correct for the bias.

This sort of bias typically appears when there's a decision being made at some point to determine whether a person should be included in a test group or not. The canonical example is when a doctor is choosing whether to treat an individual with medicine -- it is generally agreed upon that randomly assigning medicinal treatments is unethical, but we need at least pseudo-random data to predict if someone will get better once given the medicine. Or, if you prefer a less daunting example, such bias correction could also applied in correcting survey data, where we might want to correct our responses to account for the probability of response.

To address this, we need to fill in the unobserved gaps (e.g. what would happen to non-responders in a survey or individuals who aren't treated with the medicine). Two popular methods of doing so are:
1. **Outcome estimation:** Model the outcome $Y$ very, very well, and use this model to guess the unobserved responses.
1. **IPTW (Inverse Probability of Treatment Weighting):** A.k.a. propensity score matching. Model the probability of treatment and correct your observed values $Y_i$ by this correction.

Alas, don't feel as if you need to choose between the two! As we will soon discuss, there is a third method known as **doubly robust estimation**, which incorporates both of the above techniques in such a way that as long as we get one of them right, we end up with an unbiased estimator.

*Much of the following comes from [Funk et al. (2011)](https://academic.oup.com/aje/article/173/7/761/103691) and a very nice [U.Penn coursera lecture](https://www.coursera.org/lecture/crash-course-in-causality/doubly-robust-estimators-hZjgB) on the subject. If you're itching to look at this from another angle, check either of those resources out.*

Let's begin by going through each of the two estimators described above...

## Outcome estimator

An outcome estimator $M$ can be used as follows:

$$E_{R}(Y) = \frac{1}{n} \sum \left( A_i Y_i + (1-A_i) M(X_i)\right)$$

where $M$ is a model $M(X) = E(Y\|A=1, X)$. I'm using the notation borrowed from the causal inference literature, but fundamentally, all this is saying is that we can obtain an unbiased estimate of $Y$ if we take values of $Y$ where we observe them, infer values of $Y$ where we don't by using the model $M$, and finally, average all of these values.

**For the estimator to be unbiased, $M$ must be correctly specified.**

## IPTW estimator

Inverse Probability of Treatment Weighting (IPTW) is the act of weighting observations by their probability of treatment $\pi$ in order to get an unbiased estimate of some outcome $Y$.

$$E_{\text{IPTW}}(Y) = \frac{\sum_i^m Y_i / \pi_i}{\sum_i^m \pi_i}$$

where $i$ sums over all observations $m$. The literature typically favors using, instead of the number of observations, the total number of individuals (observed and unobserved) $n$ and an indicator function $A$ that denotes whether or not an individual was treated or not.

$$E_{\text{IPTW}}(Y) = \frac{1}{n} \sum_i^n \frac{A_i Y_i}{\pi_i} $$

Note that both of these definitions are equivalent. **For the estimator to be unbiased, $\pi$ must be correctly specified.**


## Doubly robust estimator

We can combine both of these to obtain an estimator such that, as long as either is correct, we have an unbiased estimator. Consider the following quantity:

$$E_{DR}(Y) = \frac{1}{n} \sum_i^n \left( \frac{A_i Y_i}{\pi_i} - \frac{A_i - \pi_i}{\pi_i} M(X_i) \right) $$

Imagine what would happen if we have a good propensity score model $\pi$, but our model $M$ is incorrect. In this case, the first term remains unaffected. What happens to the second term? We are essentially asking what is the expected value of the second term?

$$E\left( \frac{A_i - \pi_i}{\pi_i} M(X_i) \right)\\
= E(A_i M(X_i)/\pi_i) - E(M(X_i)) $$

We are weighting each $M(X_i)$ by the inverse probability of treatment, so $E(A_i M(X_i) / \pi_i) = E(M(X_i))$. This quantity, therefore, goes to 0.

Similarly, we can rearrange the doubly robust estimator as follows and ask what happens when our propensity score model $\pi$ is not correct, but $M(X_i)$ is:

$$E_{DR}(Y) = \frac{1}{n} \sum_i^n \left( \frac{A_i(Y_i - M(X_i))}{\pi_i} + M(X_i) \right) $$

If our model is perfect, $Y_i = M(X_i)$, and so the first term goes to $0$, leaving us with $E(M(X_i))$. Therefore, as long as we get either our model of the treatment $\pi$ or our model of the outcome $M$ correct, our estimator will be unbiased.
