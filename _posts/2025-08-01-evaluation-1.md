---
title: "How to Report and Judge ML Model Performance: Part 1"
date: 2025-08-01
permalink: /posts/2025/08/evaluation1/
tags:
  - Evaluation
  - Statistics
  - Distributions
  - Expected vs Typical
  - Sample size
  - Sample quality
  - Standards
  - Reporting
  - Machine learning (ML)
  - Trustworthy ML
---

In my last blog post, I laid down two bold statements about how the reviewed transformer models for molecular property prediction in [our review](https://pubs.acs.org/doi/10.1021/acs.jcim.4c00747){:target="_blank" rel="noopener"}[^1] are insufficient — both in terms of novelty and benchmarking. In this and the following posts, I hope I can convince you of these conclusions.

This post will start with extremely trivial points for those who have been doing machine learning for years. However, I highly recommend you be patient and listen to a kid who has something to say and is so enthusiastic about it. Seeing the world through the eyes of a kid can sometimes be refreshing! 
<br> Worst-case scenario: if you find a student struggling with some triviality, you might gain new intuitive ways to help them out.

This post is also accompanied by a [notebook](https://github.com/Afnan-Sultan/afnan-sultan.github.io/blob/master/tutorials/How%20to%20Report%20and%20Judge%20ML%20Model%20Performance%3A%20Part%201.ipynb){:target="_blank" rel="noopener"} to reproduce the figures shown here.

> *Disclaimer*: I'm not a statistician — just a curious wanderer who stumbled upon inconsistencies in her research and is trying to patch them up.
> I'm more than happy to be guided, corrected, or pointed toward intuitive and trustworthy content — or simply to entertain a curious discussion and see what comes of it

---

## Setting the Stage

Let's start by describing the playground. Imagine that we want to train a model to predict the aqueous solubility of a molecule.

Aqueous solubility is measured in Molar unit and can range from zero (i.e., completely insoluble in water like crystallized wax) to one Molar (i.e., completely soluble in water like salt).

Since this is an imaginary scene, let’s pretend we have access to 1 million molecules with their corresponding aqueous solubility. This will constitute the population distribution of aqueous solubility (Figure 1).

<figure>
  <img src="/images/evaluation_1/population.png" alt="Figure 1" width="600"/>
  <figcaption><strong>Figure 1:</strong> Imaginary population for aqueous solubility.</figcaption>
</figure>

In real life, we don't have access to this full distribution, but rather to a small set — usually just a couple of thousand. I stayed close to reality and imagined we have access to 2,000 (2K) molecules. However, I designed it with a **uniform distribution** — the dream scenario for an ML model (Figure 2).  
<br>A uniform distribution means that each solubility range has enough samples to (hopefully) provide sufficient information for the model to learn across all ranges.

<figure>
  <img src="/images/evaluation_1/train_dream.png" alt="Figure 2" width="600"/>
  <figcaption><strong>Figure 2:</strong> A real-world sized dataset for aqueous solubility, but with a dream uniform distribution.</figcaption>
</figure>

The goal of predictive machine learning is to build an algorithm that takes these 2K molecules and finds patterns to predict whether a molecule will have a solubility of 0.2 Molar or 0.8 Molar.

The most naïve way to do this is to choose an algorithm you like, give it the 2K molecules, let it do its magic, and then wait. 
<br> One day, your chemist gets interested in a new molecule and asks you to predict its solubility. The model says it’s going to be 0.75 Molar. The chemist goes to the lab, tests it, and it turns out to be 0.8 Molar. Your model was only 0.05 Molar off[^2]. Not bad at all, right?

But wait — does this mean your model will *always* be 0.05 Molar off for new predictions?  
What if your chemist comes with another molecule, and this time the model predicts 0.57 Molar, but the actual value turns out to be 0.36 Molar? Yikes. That’s way off! Should you shun your model and never use it again?

Well, both of these cases are anecdotal. They don't tell you much about your model and might happen rarely. When you want to say *“my model’s error is `x` Molar”*, you're usually advised to report the **expected** error value.

To report the expected value, you must test your model *enough* times on new molecules to find out what that value is (Figure 3).

<figure>
  <img src="/images/evaluation_1/error_distribution.png" alt="Figure 3" width="600"/>
  <figcaption><strong>Figure 3:</strong> An example of error distribution with highlighted expected value, a high frequent value, and a very low frequent value.</figcaption>
</figure>

---

## A Slight Detour

In statistics and probability, the **expected** value is the **mean** (i.e., average (µ)) of a distribution. However, the **expected** value is not always the **typical** value. And by *typical*, I mean the value most likely to occur — also known as the **mode**.

> **Key Point**&nbsp; The *expected* (µ) and *typical* (mode) values can differ significantly depending on the shape of the distribution (Figure 4).  
> (I mean, come on—how often do you really get what you expect? 😉)

- For a **normal distribution**, the expected and typical values are the same.  
- For a **bimodal distribution**, the typical value will be **two** values (two modes), while the expected value lies in-between and occurs less frequently.  
- For a **skewed distribution**, similar to bimodal, the mode is not the same as the average.  
- For a **uniform distribution**, **every value** is typical!

<figure>
  <img src="/images/evaluation_1/expected_vs_typical.png" alt="Figure 4" width="600"/>
  <figcaption><strong>Figure 4:</strong> Illustration of expected vs typical value by distribution shape.</figcaption>
</figure>

You'll usually find model performance reported using the **average (µ)** value. This is due to an *ideal* assumption in machine learning: that a model’s performance will tend toward the average over many predictions[^3].

---

### 🧭 I Dare to Suggest: Report the Typical, Not the Expected [^4]

So here’s a challenge to the norm: instead of defaulting to the expected (µ) value only, report the typical (e.g., mode(s))[^5] — and the shape of the story behind it (e.g., whether the distribution is normal, skewed, etc.).
<br>Or, at the very least, acknowledge when the expected and typical values diverge.

At the end of the day, we want to objectively quantify our model’s behavior: when it works and when it doesn’t. Ignoring this (seemingly trivial) distinction can lead — and probably *has* led — us to draw the wrong conclusions.

---

## ... And Coming Back

After explaining the difference between expected and typical values, I will use **“typical error”** to refer to the **most probable** error a model will produce.

P.S.: In our imaginary case, where predictions are intentionally made to follow a **normal distribution**, the typical error (mode) and the expected error (µ) are the same.

So, after testing your model *enough* times on new molecules and identifying the typical value (Figure 3), the transparent language to use would be:

> *“My model’s typical error is `x` Molar after being tested on `n` new molecules.”*

But realistically, it's not practical to wait for your chemist to test `n` new molecules every time you want to evaluate your model. So, ML practitioners came up with a clever and efficient workaround: splitting the available data into two parts — one large portion for training the model, and one smaller portion that simulates the real-world testing phase.

This is called a **train–test split**, and it’s been a staple in ML workflows for good reason. It gives us a stand-in for how the model might behave on truly new data.

In our imaginary case, we have 2K molecules — this may sound like a lot, but it's actually a modest dataset by ML standards. So, we might use a typical **80:20 split**: 1600 samples to train the model, and 400 samples to test it.  
Those 400 molecules are now your surrogate chemist queries. They must be treated as if they are completely unseen and unlearned — because they stand in for the real unknown future.

Pretty smart and efficient, right?

So, after testing my model on truly new molecules, can I now say my model’s typical error is `x` Molar?  
Not quite.

---

## Standard Deviation: How Stable Is Your Model?

The typical value (here, same as expected) might be the most probable, but how often it occurs depends on the **spread** of the distribution — described by **variance (σ²)**, or more intuitively, by **standard deviation (σ)**[^6].

> **σ** measures the *average distance* between each error and the mean error.  
> A **large σ** means predictions are spread out; a **small σ** means predictions cluster closely around the **expected** value.

If two models have the same typical error, the one with **lower σ** is more *stable* and *trustworthy*. It fluctuates less and gives more **confidence** in its predictions (Figure 5).

So, a good rule of thumb:

> **Always report both the typical/expected error and the standard deviation (σ).**  
> When performance is similar across models, σ helps you pick the more stable one.

<figure>
  <img src="/images/evaluation_1/model_stability.png" alt="Figure 5" width="600"/>
  <figcaption><strong>Figure 5:</strong> Comparison of two models with the same typical value but different standard deviation.</figcaption>
</figure>

---

## Still Not Enough…

So, is this it? Is this how you judge model performance?  
Again, not quite. σ tells you how often this expected/typical value will occur, but how confident are you in this value to begin with?

To say “my model’s typical error is `x ± σ` Molar” you must have **confidence** in that `x` value. That confidence comes from **testing on a large number of new samples**, something like **tens of thousands**[^7].

Let’s say we trained our model on 1600 molecules, tested on 400, and calculated the typical error. Now imagine your chemist comes with 200 more molecules. Do you expect the same typical error?

Most likely, no. Because 400 and 200 are **small** sample sizes in ML.  
Sometimes, the 200 new molecules will resemble the training set — the model performs great. Other times, they’ll be drastically different — performance drops (Figure 6).

<figure>
  <img src="/images/evaluation_1/sampling_uncertainty.png" alt="Figure 6" width="600"/>
  <figcaption><strong>Figure 6:</strong> Comparison of model's performance when tested on a large dataset of 10K molecules, compared to 10 small datasets each consisting of 200 molecules. The performance of the small sets fluctuates noticeably around the model's true performance.</figcaption>
</figure>

So we come **full circle**: reporting a typical error from a small test set is **not enough** to trust it — just as reporting the error on **one molecule** wasn’t enough either.

---

## Wrapping Up

Let’s summarize:

- A single prediction’s error ≠ model’s typical error.  
- The expected error (µ) can differ from the typical value (mode) depending on the distribution shape.  
- **Standard deviation (σ)** tells you how *stable* the model is.  
  - If two models have the same typical error, the one with **lower σ** is **more stable** and **more reliable**.  
- A large, representative test set is the **best** way to estimate a model’s typical/expected performance.  
  - A small test set (e.g., 200) only gives an **estimate** of the true typical/expected error, and is subject to the representative quality of the contained molecules.

But... what if you **don’t have access** to such large test data (which is the default case in our field)?  
Is all hope lost?

The answer is: **no** — and this time, that’s good news.  
But explaining why will make this post too long...

**So, see you in the next one!**

---

[^1]: Publicly accessible pre-print version [here](https://arxiv.org/abs/2404.03969){:target="_blank" rel="noopener"}

[^2]: The difference between predicted and true value is usually referred to as **error** or **residual** in machine learning.

[^3]: This assumption is grounded in the **law of large numbers**, which states that with a sufficiently large number of predictions, the average error will converge to the expected value (mean) — provided the data are independent and identically distributed (i.i.d.).

[^4]: This is a temporary challenge, to be honest. Later, I will talk about how to proceed when your expected and typical are different. Because this will be telling you *where to look*, rather than *how to report*!

[^5]: We’re explicitly discussing an **ideal** scenario in which the distribution is well-behaved, making the mode a reliable descriptor of what is “typical.”

[^6]: **Variance (σ²)** ends up in squared units (e.g., Molar²) because each error is squared; taking the square-root brings you back to the original units, giving the **standard deviation (σ)**. However, variance is the de facto spread descriptor because it is additive and differentiable — two traits calculus and probability love.

[^7]: 10K is an illustrative ballpark. The precise requirement depends on how narrow a confidence interval you need and how representative the new samples are.
