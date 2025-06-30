---
title: "How to Report and Judge ML Model Performance: Part 1"
date: 2025-08-01
permalink: /posts/2025/08/evaluation1/
tags:
  - Machine learning (ML)
  - Evaluation
  - Standards
  - Reporting
  - Statistics
  - Distributions
  - Expected vs Typical
---

In my last blog post, I laid down two bold statements about how the reviewed transformer models for molecular property prediction in [our review](https://pubs.acs.org/doi/10.1021/acs.jcim.4c00747)[^1] are insufficient, both in terms of novelty and benchmarking. In this and the following posts, I hope I can convince you of these conclusions.

This post will start with extremely trivial points for those who have been doing machine learning for years. However, I highly recommend you be patient and listen to a kid who has something to say and is so enthusiastic about it. Seeing the world through the eyes of a kid can sometimes be refreshing! Worst case scenario, if you find a student struggling with some triviality, you might gain new intuitive ways to help them out.

This post will also be accompanied by a notebook to reproduce the figures shown here — you can find it [here]().

---

## Setting the Stage

Let's start by describing the playground. We’ll use an **imaginary dataset** of 2000 (2k) molecules tested for aqueous solubility (Figure 1).  
Because it's imaginary, I designed it with a **uniform distribution** — the dream scenario for an ML model. A uniform distribution means that each solubility range has enough samples to (hopefully) provide sufficient information for the model to learn across all ranges.

![image]()

The goal of predictive machine learning is to build an algorithm that takes these 2k molecules and finds patterns to predict whether a molecule will have a solubility of 0.2 nM or 5 nM.  

The most naïve way to do this is to choose an algorithm you like, give it the 2k molecules, let it do its magic, and then wait. One day, your chemist gets interested in a new molecule and asks you to predict its solubility. The model says it’s going to be 2 nM. The chemist goes to the lab, tests it, and it turns out to be 2.2 nM. Your model was only 0.2 nM off[^2]. Not bad at all, right?

But wait — does this mean your model will always be 0.2 nM off for new predictions?  
What if your chemist comes with another molecule, and this time the model predicts 4.2 nM, but the actual value turns out to be 0.1 nM? Yikes. That’s way off! Should you shun your model and never use it again?

Well, both of these cases are anecdotal. They don't tell you much about your model. They could be extreme outcomes that happen rarely. When you want to say *“my model’s error is `x` nM”*, you are usually advised to report the **expected** error value. 

To report the expected value, you must test your model *enough* times on new molecules to find out what that value is.

---

## A Slight Detour

In statistics and probability, the **expected** value is the **mean** (i.e., average (µ)) of a distribution. However, the **expected** value is not always the **typical** value. And by *typical*, I mean the value most likely to occur — also known as the **mode**.

> **Key Point**&nbsp; The *expected* (µ) and *typical* (mode) values can differ significantly depending on the shape of the distribution. <br> (I mean, come on—how often do you really get what you expect? 😉)

- For a **normal distribution**, the expected and typical values are the same.  
- For a **bimodal distribution**, the typical value will be **two** values (two modes), while the expected value lies in-between and will occur less frequently.  
- For a **skewed distribution**, similar to bimodal, the mode is not the same as the average.  
- For a **uniform distribution**, **every value** is typical!

![image]()

You'll usually find model performance reported using the **average (µ)** value. This is due to an *ideal* assumption in machine learning: that a model’s performance will tend toward the average over many predictions.[^3]

---

### 🧭 I Dare to Suggest: Report the Typical, Not the Expected [^4]

So here’s a challenge to the norm: instead of defaulting to the expected (µ) value, report the typical (e.g., Mode(s))[^5] — and the shape of the story behind it (e.g., the distribution is normal, skewed, etc.). Or, at the very least, acknowledge when the expected and typical values diverge.

At the end of the day, we want to objectively quantify our model’s behavior: when it works and when it doesn’t. Ignoring this (seemingly trivial) distinction can lead (and probably has led) us to draw wrong conclusions.

---

## ... And Coming Back

After explaining the difference between expected and typical values, I will use **“typical error”** to refer to the **most probable** error your model will produce. 

P.S.: in our imaginary case, where predictions are intentionally made to follow a **normal distribution**, the typical error (mode) and the expected error (µ) are the same. 

So, after testing your model *enough* times on new molecules and identifying the typical value, the transparent language to use would be:

> *“My model’s typical error is `x` nM after being tested on `n` new samples.”*

![image]()

But realistically, it's not practical to wait for your chemist to test `n` new molecules every time you want to evaluate your model. So, ML practitioners came up with a clever and efficient workaround: splitting the available data into two parts—one large portion for training the model, and one smaller portion that simulates the real-world testing phase.

This is called a **train–test split**, and it’s been a staple in ML workflows for good reason. It gives us a stand-in for how the model might behave on truly new data.

In our imaginary case, we have 2k molecules—this may sound like a lot, but it's actually a modest dataset by ML standards. So we might use a typical **80 : 20 split**: 1600 samples to train the model, and 400 samples to test it.  
Those 400 molecules are now your surrogate chemist queries. They must be treated as if they are completely unseen and unlearned—because they stand in for the real unknown future.

Pretty smart and efficient, right?

![image]()

So, can you now say your model’s typical error is `x` nM?  
Not quite.

---

## Standard Deviation: How Confident Are You?

The typical value might be the most probable, but how often it occurs depends on the **spread** of the distribution — described by **variance (σ²)**, or more intuitively, by **standard deviation (σ)**.[^6]

> **σ** measures the *average distance* between each error and the mean error.  
> A **large σ** means predictions are spread out; a **small σ** means predictions cluster closely around the typical value.

If two models have the same typical error, the one with **lower σ** is more *stable* and *trustworthy*. It fluctuates less and gives more **confidence** in its predictions.

So, a good rule of thumb:

> **Always report both the typical error and the standard deviation (σ).**  
> When performance is similar across models, σ helps you pick the more stable one.

![image]()

---

## Still Not Enough…

So, is this how you judge model performance?  
Again, not quite.

To say “my model’s typical error is `x ± σ` nM” you must have **confidence** in that value. That confidence comes from **testing on a large number of new samples**, ideally in the **tens of thousands**[^7].

Let’s say we trained our model on 1600 molecules, tested on 400, and calculated the typical error. Now imagine your chemist comes with 200 more molecules. Do you expect the same typical error?

Most likely, no. Because 400 and 200 are **small** sample sizes in ML.  
Sometimes, the 200 new molecules will resemble the training set — the model performs great. Other times, they’ll be drastically different — performance drops.

So we come **full circle**: reporting a typical error from a small test set is **not enough** to trust it, just as reporting the error on **one molecule** isn’t enough either.

![image]()

---

## Wrapping Up

Let’s summarize:

- A single prediction’s error ≠ model’s typical error.  
- A small test set (e.g., 400) gives an **estimate** of the true typical error — better than one point, but still imperfect.  
- A large, representative test set (e.g., 10k) is the **best** way to estimate a model’s typical performance.  
- **Standard deviation (σ)** tells you how *confident* you can be in that typical value.  
- If two models have the same typical error, the one with **lower σ** is **more stable** and **more reliable**.

But what if you **don’t have access** to such large test data?  
Is all hope lost?

The answer is: **no** — and this time, that’s good news.  
But explaining why will make this post too long...

**So, see you in the next one!**

---

[^1]: Publicly accessible pre-print version [here](https://arxiv.org/abs/2404.03969)

[^2]: The difference between predicted and true value is usually referred to as **error** or **residual** in machine learning.

[^3]: This assumption is grounded in the **law of large numbers**, which states that with a sufficiently large number of predictions, the average error will converge to the expected value (mean) — provided the data are independent and identically distributed (i.i.d.).

[^4]: This is a temporary challenge, to be honest. Later, I wil talk about how to proceed when your expected and typical are different. Cuz this will be telling you where to look, rather than how to report!

[^5]: We’re explicitly discussing an **ideal** scenario in which the distribution is well-behaved, making the mode a reliable descriptor of what is “typical.”

[^6]: **Variance (σ²)** ends up in squared units (e.g., nM²) because each error is squared; taking the square-root brings you back to the original units, giving the **standard deviation (σ)**. However, variance is the de facto spread descriptor because it is additive and differentiable—two traits calculus and probability love.

[^7]: *10k is an illustrative ballpark. The precise requirement depends on how narrow a confidence interval you need and how representative the new samples are.*

