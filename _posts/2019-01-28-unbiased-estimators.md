---
layout: post
title: An introduction to unbiased [and doubly robust] estimators
tags: machine-learning causal-inference
---

Often, data collection cannot be completely random -- e.g. in clinical trials, where it would be unethical to randomly treat people with medicine, or in online surveys, where response cannot be guaranteed. In such cases, data can be biased, so any inferences drawn or machine learning models built from this data will not generalize well to the overall population. This is where **unbiased estimation** can come in, in which small adjustments are effectively made to the dataset to make it more representative of a random sample.

This post will introduce a couple kinds of unbiased estimation (1. inverse probability of treatment weighting and 2. outcome modeling) towards the ultimate goal of introducing the doubly robust estimator, which combines both of these things, but can be a little confusing.

## An introduction to bias, by way of example

First, let's discuss bias with an example to make the concept more tangible. Say I want to know how All People (defined as all people, all over the world) would answer the following question:

> **Is Robert the best?**

To answer this, I conduct a small survey of 100 people. Of these 100, 90 answer "Yes". From this data, we do two things: 1. we conclude that 90% of people think "Robert is the best"; 2. we build a machine learning model from this data, and it similarly predicts "Yes" about 90% of the time for All People.

But there are potential problems here. As much as I want to believe that 90% of people truly think I am the best, there's a question here: **who did I survey?**

If I surveyed 100 randomly sampled people from all over the world, it might be reasonable to generalize my results to All People. But if instead I surveyed 90 friends who answered "Yes" and 10 strangers that said "No", the average response of this group of people is biased with respect to the whole world, because I am not friends with 90% of the world's population. Moreover, any machine learning model built on this data would minimize error disproportionately amongst my friends (since they make up 90%) of the data. In fact, we could probably make a much better predictor for the general population from this data if we just always guessed "No", since all 10 strangers said "No"!

Loosely speaking, the potential problem here is **sample bias**: *data that we're might not actually represent the truth that we care about*. And this can affect both:

1. Raw estimates of the percentage of "Yes"es.
2. Machine learning models built from the data.

## A more careful explanation of bias (and some jargon + notation)

Both the estimated % of "Yes"s and the machine learning model above are examples of **estimators**, where an estimator $\hat{Y}$ is anything that makes an estimate of some parameter $Y$. In the simplest case, $\hat{Y}$ can be a sample mean or median, where $Y$ is the true population mean or median (e.g. the percentage of "Yes"es above). In machine learning problems, $Y$ is usually just the label we're trying to predict, and $\hat{Y}$ is the output of the machine learning model.

An estimator $\hat{Y}$ of a parameter $Y$ is **biased** if:

$$\text{Bias} \equiv E[\hat{Y} - Y] \neq 0$$

Bias is the *expected value of the difference between an estimator, $\hat{Y}$, and the true value that it's trying to predict $Y$*.

Before we move on, I'd like to clarify a point I once had some trouble with: **the estimand $Y$ must be defined over some domain**. Usually, this is taken to be all observations, but note that it doesn't need to be. Using an estimate of $0.9$ for the fraction of people that think I'm awesome is very biased over all people, but it may not be so biased if I looked at, say, readers of this blog post (particularly if the only people reading this are my friends).

## Correcting bias

If we want to construct an unbiased estimator out of a biased dataset, we have two options:
1. **Outcome estimation:** Model the outcome $Y$, and use this model to guess the unobserved responses. This is also known as the "Direct Method."
1. **IPTW (Inverse Probability of Treatment Weighting):** Model the probability of observing a response $\pi$ and correct your observed values $Y$ by this prediction.

In the machine learning world, **Outcome estimation** is actually usually the thing we're trying to do anyway, so (1) is like hoping that your model built on biased data just generalizes well out of the box. **IPTW** is a way of reweighting your less commonly occurring data points so that their proportion when averaging or calculating loss functions will be reflective of the population you care about. However, this can be problematic if you don't have *any* observations from certain kind of people (when you *do* have some observations from all kinds of people, it is commonly referred to in the literature as having **overlap**).

Let's begin by going through each of the two estimators described above. After we do, I'll introduce a third method known as **doubly robust estimation**, which incorporates both of the above techniques in such a way that as long as we get one of them right, we end up with an unbiased estimator.

### Outcome model

An outcome model $M$ can be used to obtain an unbiased estimator $\hat{Y}_{\text{OM}}$ as follows:

$$\hat{Y}_{\text{OM}} = A Y + (1-A) M(X)$$

where $M(X) = E[Y\|A=1, X]$ is a model and $A$ is an indicator function that denotes whether or not an individual is observed or not. I'm using the notation borrowed from the causal inference literature, but fundamentally, all this is saying is that we can obtain an unbiased estimate of $Y$ if we take values of $Y$ where we observe them and infer values of $Y$ where we don't by using the model $M$.

**For the estimator to be unbiased, $M$ must be correctly specified.**

In the context of the "Is Robert the best?" survey, this amounts to training a classifier on our biased data which just predicts "Yes" or "No", and hoping for the best. If we get lucky, this model may even generalize well. For example, suppose we had a nice binary feature "Robert's friend" that cleanly correlates with the response to this survey. In such a case, creating a model off the biased data might work well enough to serve as an unbiased estimate for All People.

### IPTW estimator

Inverse Probability of Treatment Weighting (IPTW) is the act of weighting observations by their probability of treatment $\pi$ in order to get an unbiased estimate of some outcome $Y$.


$$\hat{Y}_{\text{IPTW}} = \frac{A Y}{\pi} $$

**For the estimator to be unbiased, $\pi$ must be correctly specified and can't ever be $0$.**

Once again, in the context of the "Is Robert awesome?" survey, let's assume we had $90$ "Yes"s (from my friends) and $10$ "No"s (from strangers). In this case, $\pi$ would be a model that predicts the probability of being surveyed. If we use our binary feature "Robert's friend" again, we'll probably find that $\pi(\text{Robert's friend} == \text{True}) = 1$, and $\pi(\text{Robert's friend} == \text{False}) \simeq 10^{-9}$. We would then, consequently, count each of my non-friends $10^9$ times so any averages we calculate reflect the true distribution. In a machine learning context, we could then use this reweighted data to train a model (but we have to be careful to reweight correctly everywhere, including in our loss function).

## Doubly robust estimator

We can combine both of these to obtain an estimator such that, as long as either is correct, we have an unbiased estimator. Consider the following quantity:

$$\hat{Y}_{DR} = \frac{A(Y - M(X))}{\pi} + M(X) $$

Consider first the case where $\pi$ is a perfect model. In this case, we are weighting each $M(X)$ by the inverse probability of being surveyed, so in expectation, $\pi$ should perfectly reweight observed $A$'s, giving $E[A (Y - M(X))/ \pi] = E[Y - M(X)]$. We therefore have $E[\hat{Y}_{\text{DR}}] = E[Y]$.

On the other hand, if our model $M$ is perfect, $Y = M(X)$, and so the first term goes to $0$, leaving us with $E[\hat{Y}_{\text{DR}}] = E[M(X)] = E[Y]$.

Therefore, **as long as we get either our model of the treatment $\pi$ or our model of the outcome $M$ correct, our estimator will be unbiased**.

## A recursively robust estimator

We can actually repeat the doubly robust process recursively. To do this, $\hat{Y}_{\text{DR}}$ can simply be used as the model $M(X)$ into another doubly robust estimator, giving us another try at modeling $\pi$. Done once, this looks something like this:

$$\hat{Y}_{\text{RR}, 2} = \frac{A\left(Y - \left(\frac{A(Y - M(X))}{\pi_1} + M(X)\right)\right)}{\pi_2}  + \left(\frac{A(Y - M(X))}{\pi_1} + M(X)\right) $$

Written out, it quickly becomes horrendous, but if you go through the work of checking it, you might notice that we now get two chances to attempt the IPTW model ($\pi_1$ and $\pi_2$), and if either is correct in expectation, $\hat{Y}_{\text{RR},2}$ should be unbiased. This can be repeated forever! Unfortunately, in practice, nesting models like this may not be that useful. The primary reason why constructing a perfect model $\pi$ is difficult is because there are often *unobserved* confounding variables, and no number of nested models will address this.


## Final comments

So why do we even want to do this?

As I've said, these techniques can certainly be useful in obtaining an unbiased estimate for reporting purposes or, with some care, in training an unbiased model from biased data. But I would be remiss if I did not mention how unbiased estimation can be useful for **causal inference**.

In causal inference, we want to parse the effect of a treatment. Without unbiased estimates of an outcome with respect to both a treated group and a control group, any inferences made on the basis of your data could be incorrect! And this is often the case -- a lot of inferences are drawn from *observational* rather than experimental data. If, for example, I wanted to know the effect of giving $5 to my survey-takers, if my friends' responses didn't change but strangers' responses did, bias that stems from incorrect proportions of friends and strangers could cause me to be very inaccurate in my estimate of the *causal effect* of bribing survey-takers (in this case, I'd think the effect small if I mostly surveyed my friends, who would take my survey as a favor, not because I gave them money). Or, more concrete examples can be found in fields like advertising or medicine, we could end up wasting money, or worse, missing out on powerful medicines!

*The above post was loosely inspired by [Funk et al. (2011)](https://academic.oup.com/aje/article/173/7/761/103691) and a very nice [U.Penn coursera lecture](https://www.coursera.org/lecture/crash-course-in-causality/doubly-robust-estimators-hZjgB) on the subject. If you're itching to look at this from another angle, check either of those resources out. I have not found a source for the infinitely robust estimator. Also, if you are still skeptical that sample bias is a serious problem, check out this fun example involving [political polling](https://fivethirtyeight.com/features/the-polls-were-skewed-toward-democrats/).*
