---
layout: single
title:  "How to Report and Judge ML Model Performance: Part 4"
header:
  teaser: "unsplash-gallery-image-2-th.jpg"
categories: 
  - Jekyll
tags:
  - Metrics 
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

In the last two blogs, we have been asking how to report -with confidence-the expected performance of our model. And we have been discussing one way of reporting, which is reporting the error (i.e., residuals). In this blog post, I will take a brief dive on different metrics to report model performance, what each metric is telling us, and how to maximize the benefit of using one or more such metrics.

Let's start by fixing some terminology. The model we have been talking about so far is trained as a **regression** model. In regression, the model sees data points, characterized by some features, and come with corresponding continuous values. The goal of a regression model is to figure out with some combinations of these features, what would be the expected value of a corresponding data point. 

Let's solidify this with an examples that is readily intuitive for anyone, Houses.

Let's say that you have the prices list of 10,000 houses located in Germany, each house is characterized using some features like city, size, number of bedrooms, facilities, etc. The goal of your regression model is to predict the expected price of a new house based on its features. For example, if the new house has 3 bedrooms, 100 square meter and located in Saarland, then your model have seen/understood that houses with these features tend to have a price around 500K euros.

So, given the features of the house, the model was able of telling you, on average (i.e, expected), what would this house's price be.

How can you be confident in this predicted value? That's what we have been discussing in the last blog posts! and we have been doing so by quantifying the model performance with the error (i.e., residuals) metric.

# Error (i.e., Residuals)

The equation for residuals is 

$Residual = Predicted - True$. 

It simply tells you whether the model has predicted a new value correctly (i.e., residual = 0), it overestimated it (i.e., residual > 0), or it underestimated it (i.e., residual < 0) (Figure 1).

Given a test set of `n` data points, the residuals are expected to behave as a normal distribution with mean 0. The reason this is expected is that for a model to perform better than randomly guessing, at least 50% of the predictions should be very close to the true value (i.e., predicted - true ≈ 0). If the expected value of a model's residual is not 0, this is already telling you that there is something wrong with the model, the data, or the features.

Assuming that everything is fine and the model's expected residual is 0, we move to looking at the standard deviation (σ). Of course, the smaller the σ the better cuz this will mean that your model's residuals are too close to 0, but σ, and consequently, the range of the residuals distribution can tell you whether you should be satisfied with your model or not. 

Let's go back to our aqueous solubility example. It's a dataset of solubility values that range from 0 to 1 Molar. If a model's residuals σ is 0.05, this means that for ~ 65%[^1] of the data, the model will be ~5% off the true solubility value. Now, I am not a chemist, and I do not know if 0.05 Molar is an acceptable error rate for aqueous solubility or not, but I know, using my human intuition, that 5% is a very low percentage, and I would be more than happy to see my model showing this error rate. 

In another example, if the model's residual σ is 0.15, then for 60% of the data, my model would be ~30% off. Now, 30% is starting to hit an alarm for me, it is telling me that there is still a nice room for improvement, and my next step would be to see how.

If a model's residuals σ is 0.3 (60%), then the model is failing catastrophically.

Note that the percentage calculated from σ is dependant of the dataset range (here, it's 1). The equations is

$(σ / data range) * 100$

This contextualizes the error in terms of percentage rather than the original unit. The original unit would be more intuitive for the experts (e.g., our chemists), while % would be intuitive for anyone.

So, to wrap up the residuals metric, it gives you an idea about how often your model predicts correctly, and whether this is an acceptable performance. The metric is reported in the original unit of the dataset, and can be contextualized in percentage for easier interpretation. The residuals distribution is expected to behave as a normal distribution with mean 0, if not normal or mean not 0, there is something wrong that you need to figure out!

The best way to report residuals is µ ± σ alongside ensuring the normality of the distribution shape.

# Mean Absolute Error (MAE)

While the residuals metric reports both positive and negative errors (i.e., over- or under-estimated predictions), the MAE metric first converts all negative errors to positive (hence, the absolute word in the name), and reports the average of this absolute error. 

The idea of using MAE rather than residuals is when one is not interested in the direction of the error, but rather the magnitude. For example, your chemist will be equally dissatisfied if your model's predictions for solubility were higher than the true value, as well as if they were lower.

The distribution of MAE is supposed to be skewed because it's the folded distribution of the residuals' distribution! So, we already know that the average (i.e., expected) of this distribution is not the typical, but rather, the average of making big mistakes (the avergae error is already supposed to be 0 as seen in the residuals distribution)

# Mean Squared Error (MSE) and Root Mean Squared Error (RMSE)


# Coefficient of determination ($R^2$) and Adjusted ($R^2$)




[^1]: More intuition on this in a later post 