---
layout: single
title:  "How to report and judge an ML model performance."
header:
  teaser: "unsplash-gallery-image-2-th.jpg"
categories: 
  - Jekyll
tags:
  - Machine learning (ML)
  - Evaluation
  - Standards
---

In my last blog post, I laid two bold statements about how current proposed transformer models for molecular property prediction are not sufficient, both in novelty and benchmarking. In this post, I hope I will convince you with these conclusions. 

The post will start extremely trivial, for those who have been doing machine learning for years, they will probably wanna gauge their eyes out. But Just have the patience to listen to a kid who has something to say and feels so enthusiastic about. Seeing the world from the eyes of a kid can be refreshing sometimes! worst case scenario, you will be able to understand when a student is struggling with some triviality, where this confusion might be coming from!

This post will also be accompanied by a notebook to run and produce the figures to be shown here, you can find it [here]()

Let's start by describing the playground. We will have an imaginary dataset of 2000 molecules tested for aquas solubility (Figure 1). 
<br> Because it's an imaginary dataset, I made it of the dream distribution for an ML model, uniform. A uniform distribution means that for each solubility range, there is enough samples to -hopefully- provide enough information for the model to extract per this range.

![image]()

The goal of predictive machine learning is to have an algorithm that can take these 2000 molecules, catch on some patterns that will determine when a molecule will have a solubility of 0.2 nM or 5 nM. 
<br> The most naive way to do so, is to select an algorithm of your liking, give it the 2K molecules, let it do it's magic, then wait for your chemist to be interested in a new molecule, asks you to predict its solubility, the model says it's gonna be 2 nM, your chemist goes to the lab tests it experimentally, and it turns out to be 2.2 nM. Your model was only 0.2 nM off, that is not bad at all, right!

But wait a minute, does this mean that your model will always be 0.2 nM off when predicting a new molecule? 
<br> Well, your chemist came with a new molecule, the model predicted 4.2 nM, but the chemist found it to be 0.1 nM. Woooh! that's not cool at all!! this is soo off!! Shall you chun your model and never use it again? 

Both cases do not tell you much about your model, they can be extreme cases that can only happen rarely. When you want to say "my model error estimate is `x` nM" You want to report the `expected error value. The value that will happen most of the time. 
<br> For a normal distribution, this value is the "average". For a bimodal distribution, this value will be two values actually, which are the two modes of the distribution. For a skewed distribution, this value will again be the mode. And so on. The expected value to report for your model will depend on the distribution shape of you model's error. 

So, to report the expected performance of your model, you need to have tested it enough times on new molecules, find out what this expected value is, and then say, "my model is expected to make an error of `x` nM after being tested on `n` new samples."

![image]()

Now, it is not practical at all to wait for your chemist to figure out the next `n` molecules of interest for your model to predict and for the chemist to verify. So, what the smart ML people ended up doing was splitting a dataset into two parts, a huge part which will be used for training, and a small part which will be used for testing. 
<br> We have 2K imaginary samples in our dataset, the split usually happens 8:2 train-test-ratio when the sample is not that huge. So, if we split our 2K molecules into 8:2, we end up with 1600 samples for training, and 400 samples for testing. 
<br> Now, remember, these 400 samples will be treated the same way as you would treat the new molecules from your chemist. They are samples that your model should never see or learn from. So, you literally set them aside completely, do nothing with them until your model finishes training, and only then would you show the molecules to the model, ask it to predict the solubility values, then compare your model's predictions to the true value that you already have. This is pretty smart and efficient stuff, ha!

![image]()

So, is this it? can you now say with confidence that your model is expected to make an error of `x` nM after being tested on 400 molecules? 
<br> Unfortunately, no!

The expected value is indeed the most probable value, but the spread of your distribution has an influence on this probability. The spread of a distribution is described by its variance (standard deviation (SD) for more intuitive description). What SD tells you is that, whether your most probable value will be happening 90% of the times, or 60% of the times. 
<br> And of course it makes a difference to know this information. If you have two models that give the same expected value, but one model has a much narrower SD than the other, then you would go with the model with smaller SD, because this would be considered a "stable" model. The predictions will not be fluctuating that much, and you will have more confidence in your model.

So, a rule of thumb, always report the expected and SD values of your model performance distribution. because when two models have the same, or very close, expected performance, it is the SD that will help you decide which model to trust more.

![image]()

So, is this it? is this how I report and judge my model performance? 
<br> Again, unfortunately no! It is a very honest and transparent way, but to be able to say with confidence "my model is expected to make `x ± sd` nM and this is a better performance than this other model", you need to be **confident** about this expected performance of your model! 
<br> this confidence is obtained when you have tested your model on a huge number of new molecules, preferably in 10s of thousands...