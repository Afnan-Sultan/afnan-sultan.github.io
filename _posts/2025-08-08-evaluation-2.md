---
title: "How to Report and Judge ML Model Performance: Part 2"
date: 2025-08-08
permalink: /posts/2025/08/evaluation1/
tags:
  - Central Limit Theorem (CLT)
  - Evaluation
  - Statistics
  - Distributions
  - Sample size
  - Sample quality
  - Standards
  - Reporting
  - Machine learning (ML)
  - Trustworthy ML
---


In the last post, we saw why it's important to report a model's performance distribution more mindfully. We discussed how what is *typical* (e.g., the mode) can differ from what is *expected* (i.e., the mean (µ)), and how standard deviation can quantify our model's stability. However, we also noted that one distribution from a small test set is not guaranteed to give an overview of the model's true performance. In this post, we'll discuss how to move forward from this bottleneck.

This post is also accompanied by a [notebook](){:target="_blank" rel="noopener"} to reproduce the figures shown here.

> *Disclaimer*: I'm not a statistician — just a curious wanderer who stumbled upon inconsistencies in her research and is trying to patch them up.
> I'm more than happy to be guided, corrected, or pointed toward intuitive and trustworthy content — or simply to entertain a curious discussion and see what comes of it

---


# Describing the Problem

We have trained a model, and the question we are trying to answer is:

> *What should I expect the typical/expected error of this model to be when tested on new molecule(s)?*

The problem is that the expected/typical performance of a single small test set is not guaranteed to represent the model’s true expected error. This could be circumvented by significantly increasing the number of data points in the test set, as shown by the 10K Gold Standard example in Figure 1.

<figure>
  <img src="/images/evaluation_2/sampling_uncertainty.png" alt="Figure 1" width="600"/>
  <figcaption><strong>Figure 1:</strong> Comparison of model performance when tested on a large dataset of 10K molecules, versus 20 small datasets each consisting of 200 molecules. The small sets fluctuate, while the true expected value is around the average of these fluctuations.</figcaption>
</figure>

Unfortunately, we rarely have access to large test sets in our field. So, this solution is not feasible. What *is* feasible is actually emerging from the trend in the same figure!

Looking deeper into Figure 1, we can see that while each sample does not capture the Gold Standard’s expected value individually, plotting the means of all these samples approximates it very well (Figure 2)!

<figure>
  <img src="/images/evaluation_2/sample_means_vs_population.png" alt="Figure 2" width="600"/>
  <figcaption><strong>Figure 2:</strong> The expected value of the means of 10 small sample datasets approximates the expected value of the true Gold Standard distribution. This is an illustration of how the Central Limit Theorem (CLT) works.</figcaption>
</figure>

What we just observed is known as the effect of the **Central Limit Theorem (CLT)**. CLT states that, if you want to approximate the expected value (i.e., mean) of a distribution, you just need to sample small, independent, and identically distributed (i.i.d.) subsets from this distribution, calculate the mean of each sample, and the distribution of these means will narrow down the likely range of the true expected value.

# The Solution to the Problem

So, to exploit CLT, we need to search for small test samples that are:

1. **Independent** (e.g., the molecules in one sample are not the same or related to those in the same or another sample)
2. **Identically Distributed** (e.g., all molecules are measured for the same property, like aqueous solubility—not mixed with other properties like lipophilicity or melting point)

The **independence** condition is crucial. If your molecules are from the same family or share characteristics, the model may make similar predictions for all of them, and the error distributions would cluster in one region of the model's performance space (Figure 3). This fails to faithfully cover the full range of the model's true behavior.

<figure>
  <img src="/images/evaluation_2/merged_dependant_distributions.png" alt="Figure 3" width="600"/>
  <figcaption><strong>Figure 3:</strong> When samples are dependent, the model reports errors concentrated in specific regions, thus failing to capture the full distribution of possible performance outcomes.</figcaption>
</figure>

The **identically distributed** condition is essential because it's tied to the actual question we're trying to answer. If our problem is "We want to predict aqueous solubility using molar units that range from 0 to 1", then we must not test it on molecules that would fall outside these conditions.

To give a naive example: imagine test sets that actually belong to a lipophilicity dataset rather than aqueous solubility. These may be measured in LogD and range from 1 to 2. If our model is trained to predict solubility values between 0 and 1 Molar, then comparing its predictions to LogD values will always result in a large error. Even if the model predicts “1” and the true value is “1”, that would just be a coincidence!

<figure>
  <img src="/images/evaluation_2/merged_OOD_distributions.png" alt="Figure 4" width="600"/>
  <figcaption><strong>Figure 4:</strong> When samples are out-of-distribution (OOD) relative to the original training data, the model's errors no longer reflect its true performance.</figcaption>
</figure>

The example is simplistic, because nobody would likely mix completely unrelated properties. However, in real-life, this issue manifests subtly—by mixing different *ranges* of the *same* property. This is known as an **Out-of-Distribution (OOD)** problem.

In our hypothetical case, we assume the full population of aqueous solubility ranges from 0 to 1 Molar. But in practice, datasets often focus on specific ranges. For example, drug solubility is typically measured in micro-molar (µM), and the relevant range might be 1 µM to 1 mM.

If a model is trained to predict values within this narrow range, it should not be expected to perform accurately on molecules outside it.

> **The model knows what it has seen. It does not know what is unknown to it!**

So, here we are with our solution: to estimate a model’s true performance, we need access to multiple small test sets that are **independent** and **identically distributed**.

Can we access these?

If yes, then hooray! We've reached the end of the blog. Test your model on these small sets, plot the distribution of their means, and you've found a good approximation to the model’s true expected error.

**P.S.:** We've focused here on estimating the **expected** value (µ), because CLT applies to means. Estimating the **median**, **mode**, or **standard deviation** would require different approaches—ones I’m not yet equipped to handle!

# Another Caveat

I hope it’s becoming clear that things are never this easy! Machine learning is about machines learning from our world, and our world is *messy*! So let’s talk about the caveat to this already-challenging solution.

Even when we follow the CLT-based approach, the **characteristics of the i.i.d. samples** matter. If we revisit Figure 2, we notice that the distribution of the sample means is indeed centered around the true Gold Standard mean. However, the **spread** of this distribution—described by the **Standard Error of the Mean (SEM or SE)**—is still wide enough to make us pause.

This SEM tells us that the true expected error lies *somewhere* between the distribution’s edges. In Figure 2, this range is roughly between -0.2 and 0.2. Given that the actual mean is 0, and the dataset values lie between 0 and 1, saying “I’m 100% sure the model’s expected error lies between -0.2 and 0.2” is rather liberal.

In this example, we happen to know the true distribution, and the approximation was good. But usually, we don’t know. So how do we **increase our confidence** in our estimation?

The answer lies in understanding how **sampling** works: we collect `K` i.i.d samples of size `N`. Increasing `K` (number of samples) gives more points in the mean distribution, making the shape smoother. Increasing `N` (data points per sample) makes each sample more representative, narrowing the range of our estimate (Figure 5).

<figure>
  <img src="/images/evaluation_2/sample_means_size_vs_num.png" alt="Figure 5" width="600"/>
  <figcaption><strong>Figure 5:</strong> Increasing the sample size narrows the estimate of the model’s true expected performance. Increasing the number of samples sharpens the distribution shape (relevant for significance analysis, to be discussed in a future post).</figcaption>
</figure>

So, beyond ensuring samples are i.i.d., we should also consider **how many samples** we collect, and **how many data points** are in each.

# Wrapping Up

Let’s summarize:

- When a large test set is unavailable, we can approximate expected performance using multiple small test sets.
- These small sets must be **independent and identically distributed (i.i.d.)** to ensure reliable sampling.
- **More data points per sample** = narrower estimate of the expected value → higher confidence.
- **More samples** = smoother mean distribution → better basis for significance analysis (coming later!).

# The Problem, Again

I’m truly sorry to do this to you—but even with all we've discussed, we haven’t reached the end. Because most of us **don’t have access** to many such i.i.d. small test sets!

I’ve been working with molecular property datasets for over two years, and I still haven’t found a dataset with enough molecules for both training and this kind of testing scheme. If *you* have access to such a dataset, I would be **over the moon** if you shared it with me!

So... what can we do???

That will be the topic of the next post.

# Final Remark

Just so you’re not disappointed after two posts without reaching a final answer—I want to say that *the answer is not always what we are looking for*. The next post won’t offer a magic spell, but it will offer clever tricks and workarounds people have developed to cope with the messiness of the world.

However, being aware of the nuances discussed here and in the previous post is *crucial*. Knowing our limitations equips us to 
* Better understand where the next steps for improvement lie. 
* Use a better language when talking about our ML models. 
* Guide our chemists—and be guided by them—toward the next best practice that brings us a little closer to that ever-elusive answer.
