---
layout: single
title:  "The beginning: something is broken!"
header:
  teaser: "unsplash-gallery-image-2-th.jpg"
categories: 
  - Jekyll
tags:
  - Machine learning (ML)
  - Transformers
  - Language models
  - Evaluation
  - Molecular Property Prediction (MPP)
  - Standards
---

So, my PhD topic is to implement transformer models for toxicity prediction and molecular generation.

To be able to tackle this endeavour, I began by venturing into the wilderness—reading the papers that trained transformer models for molecular property prediction. While reading, I asked myself four key questions:

1. What is there?  
2. What is good?  
3. What is missing?  
4. What is promising?

![Intro](/images/blog_post_1/1.png)

I already had some experience with transformer models from the field of Natural Language Processing (NLP),  
so I was aware of the nuances in the architecture. As I started exploring the literature, I found that **sooo many** things had been tried independently, with each paper introducing its own tweaks. My first thought was:

> Let's put into tables what these papers did, so that we know what was done similarly and what was done differently.

And that seed eventually bloomed into our review paper on [Transformers for molecular property prediction](https://pubs.acs.org/doi/10.1021/acs.jcim.4c00747){:target="_blank" rel="noopener"}!

<figure>
  <img src="/images/blog_post_1/2.png" alt="Figure 1" width="600"/>
  <figcaption><strong>Figure 1:</strong> The review lists different implementations across the literature—spanning pre-training dataset sources and sizes, tokenization methods, positional encoding, model parameters, fine-tuning, and other finer details.</figcaption>
</figure>

So, with the tables in Figure 1, we answered the first question: **What is there?**

---

Now, we move to the second question: **What is good?**

Answering this question required a two-fold approach:

1. **True hypothesis**: If a modification to a method is proposed, prove that it improves over the unmodified method (aka, **ablation analysis**).
2. **Superior benchmarking**: If a new method is proposed, prove that it competes against simpler, pre-existing methods.

Armed with this framework, we started checking whether the proposed models satisfied both criteria.  
Starting with the "**true hypothesis**" criterion, we found that almost all papers reported their model's performance in the same way, a table (Figure 2).

<figure>
  <img src="/images/blog_post_1/3.png" alt="Figure 2" width="600"/>
  <figcaption><strong>Figure 2:</strong> Almost all papers report their performance in table format, highlighting the best number in bold.</figcaption>
</figure>

Let’s acknowledge something here: this table view is one of the **most horrific** ways to report model's performance.  
Not just because [Pat Walters keeps ranting about it](https://practicalcheminformatics.blogspot.com/2025/03/even-more-thoughts-on-ml-method.html){:target="_blank" rel="noopener"}, but because **psychology** strongly supports the criticism:

> Humans are TERRIBLE with numbers! [1–3]

Our brains are not wired to hold onto numeric information—especially **decimals**. Add a ± standard deviation or error margin, and you’ve got a recipe for confusion.

What our brains do handle well is **spatial information**. Half of our cerebral cortex is dedicated to it [4].  
Experiments have shown that visuals (e.g., bar plots with references) aid decision-making far better than numeric tables [5, 6].

I’d be more than happy to discuss the best visualization methods, but for now, I would like to make one thing clear:

> **ANYTHING** beyond table reporting is a step in the right direction.

So, because I knew that I couldn't trust myself (or others) to glean insights from raw numbers, I decided to manually[^1] 😣 extract performance numbers, put them in Excel, and start plotting them myself.

<figure>
  <img src="/images/blog_post_1/4.png" alt="Figure 3" width="600"/>
  <figcaption><strong>Figure 3:</strong> We plotted the performance tables ourselves; since each article reported performance differently, the plots# types also varied.</figcaption>
</figure>

Four of the reviewed articles provided **ablation analyses**. We used two types of plots based on how each paper reported their table of numbers —point plots (averages) and plots with error bars (standard deviations).

Now we could actually see performance differences. But interpreting the plots wasn't always trivial.

**The conclusion?**

> **Almost no new modification showed clearly superior performance over the unmodified method; further analysis is needed to claim superiority.**

Yes, it’s a bold statement—especially when most articles **claimed** their method was better.  
For the full walkthrough, check my other post _"How to report and interpret performance."_ 😉

---

Now onto the second fold: **Superior benchmarking**.

Any new model should be accompanied by an answer to the question:  
**Why should I use it?**  

This is especially crucial for models as computationally heavy and intricate as transformers. So, we looked at how these papers benchmarked their models (Figure 4):

<figure>
  <img src="/images/blog_post_1/5.png" alt="Figure 4" width="600"/>
  <figcaption><strong>Figure 4:</strong> Each article compared their model to various basic and advanced ML models.</figcaption>
</figure>

Some articles didn’t even compare against classic models like **random forests (RF)**—which are known strong baselines in our field [x–y].

Worse still, most articles presented their benchmarking in **performance tables**, again making proper judgment hard.

But **the most catastrophic issue**?  
Many compared their results not by re-running other models, but by copying performance numbers from separate publications (Figure 5).

<figure>
  <img src="/images/blog_post_1/6.png" alt="Figure 5" width="600"/>
  <figcaption><strong>Figure 5:</strong> A common practice: comparing your result to a copied number from another model, without matching testing conditions.</figcaption>
</figure>

This is unacceptable because, for a **fair comparison**, both models need to be tested on:

- the **same test sets**  
- the **same repetitions** (e.g., K-folds)  
- the **same splitting technique** (e.g., scaffold split)

We distilled how each article performed its tests and found: **there is no standard** (Figure 6).[^2]

<figure>
  <img src="/images/blog_post_1/7.png" alt="Figure 6" width="600"/>
  <figcaption><strong>Figure 6:</strong> Papers follow very different testing schemes, making comparisons infeasible.</figcaption>
</figure>

So, comparing your number to someone else’s is like this: one person tastes a Granny Smith apple, and another tastes a Fuji apple. Then, you claim that the person who tasted the Fuji apple is better at detecting sweetness. But you forget to mention that they didn’t even taste the same apple!
![comic](/images/blog_post_1/8.png)

To draw honest conclusions, we restricted our analysis to models tested under **identical conditions**.  
Again, we plotted the numbers (Figure 7).

<figure>
  <img src="/images/blog_post_1/9.png" alt="Figure 7" width="600"/>
  <figcaption><strong>Figure 7:</strong> Only 5 models reported benchmarking in a comparable way. We visualized their performance.</figcaption>
</figure>

**Conclusion?**

> **For each transformer model, there's a simpler model that performs similarly, better, or the difference is unclear without deeper analysis.**

(Again, details in the blog post _"How to report and interpret performance."_)

---

Trying to answer "**What is good?**", I hoped to find clear trends for building the next great transformer model.  
Instead, I found that most analyses weren’t sufficient to draw conclusions.

The next two questions became surprisingly easy:

- **What is missing?** → **Standards to answer what is good.**  
- **What is promising?** → **Applying such standards to existing work to decide whether it was justified!**

---

In the next blog post, I’ll walk you through one of these standards in action—*repeating an existing study* to see if it holds up.  
Spoiler alert: it didn’t! 😉

---

### References

[1] Edward Tufte: *The Visual Display of Quantitative Information*  
[2] Daniel Kahneman: *Thinking, Fast and Slow*  
[3] Gerd Gigerenzer: *Calculated Risks*  
[4] Felleman & Van Essen. *Cerebral Cortex*, 1991  
[5] Cleveland & McGill. *Journal of the American Statistical Association*, 1984  
[6] Karin Eberhard. *Management Review Quarterly*, 2023

---

[^1]: This introduces human error. The performance tables weren’t available in Excel, so I had to copy-paste (when feasible) or screenshot them, then use Excel’s image capture and proofreading. It was a *lot* of manual work—and we all know that’s not ideal. I tried my best to ensure accuracy, but no guarantees.

[^2]: Information was distilled from each manuscript and, when needed, scavenged from GitHub repositories. If something is misreported, it may be due to my mistake (apologies!) or a lack of transparency in the original article.
