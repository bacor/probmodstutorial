---
layout: default
title: Probabilistic Models of Cognition - 2nd Edition
custom_js:
- assets/js/index.js
custom_css:
- assets/css/index.css
---

<div id="header">
  <h1 id='title'>Hierarchical Bayesian Models</h1>
  <hr class='edition' />
  <span class="authors">Bas Cornelissen, based on Goodman &amp; Tenenbaum's ProbMods</span>
</div>

<br />
In this tutorial will look at hierarchical Bayesian models of categorization, using a probabilistic programming language.
Generative models can be very conveniently described and simulated in such programming languages. 
For that reason, they are a great tool for exploring probabilistic models of cognition.
The language we use here, [WebPPL](http://webppl.org) ("web people"), has partly been developed for that by [Noah Goodman](http://cocolab.stanford.edu/ndg.html) and [Joshua Tenenbaum](http://web.mit.edu/cocosci/josh.html).
As the name suggest, WebPPL is based on a *web* programming language: JavaScript. 
As a result, all code runs directly in your browser.
There is no need to install any additional software.


In their online book *[Probabilistic Models of Cognition](probmods.org/v2)*, Goodman and Tenenbaum present a wide range of probabilistic cognitive models.
The goal of this tutorial is to implement some of the hierarchical Bayesian models from that book. Parts of the tutorial come directly from the book.


**Important note:** The changes you make to the code are lost once you refresh the page. If you want save your code, copy-paste to some text editor. This should not be a problem as you don't have to code much. 

**Tip:** If you want to play around with WebPPL, you can find an editor and more examples on [webppl.org](http://webppl.org).

# Preparation
Since we have only two hours for the actual tutorial, we need to get through the basics quickly. 
That will work best if you come prepared, even though the preparations shouldn't take a lot of extra time.

1. Make sure you have read the [tutorial on Bayesian modelling by Amy Perfors et al. (2011)](http://www.sciencedirect.com/science/article/pii/S001002771000291X). Section 3 ("aquiring inductive constraints") is particularly relevant: the tutorial implements some of the models discussed there.
2. JavaScript is not R, so please read this [a very brief introduction to JavaScript]({{ site.baseurl }}/chapters/13-appendix-js-basics.html). As you will see, the syntax is not very different from R.
3. **Optionally**, if you are interested in the underlying ideas, read [this short, general introduction]({{ site.baseurl }}/chapters/01-introduction)
4. **Optionally**, if you want to be super prepared, read [this (excerpt from) the second chapter on generative models]({{ site.baseurl }}/chapters/02-generative-models.html). The latter might be particularly useful for those who have never seen any probability theory, but you should also be able to do the tutorial without reading it.
5. **Optionally**, if you want to know more about the model we will be implementing, read the first part of [Kemp, Perfors &amp; Tenenbaum (2007)](http://onlinelibrary.wiley.com/doi/10.1111/j.1467-7687.2007.00585.x/full), up to the section "modelling inductive reasoning". There is substantial overlap with section 3 in Perfors et al (2011).

<!-- If you have any questions please contact Bas or ask them during the lab. -->

# The tutorial
Probabilistic models are often best explained in simple scenarios.
In this case, a classic one: bags with coloured marbles.
In the first part of the tutorial we will be solely concerned with bags and marbles.
Starting from the very basics of probability theory we quickly work towards a hierarchical model (the so-called [Dirichlet-Multinomial model](https://en.wikipedia.org/wiki/Dirichlet-multinomial_distribution)).
The second part slightly extends the model and deals with its cognitive interpretation.

0. [A very brief introduction to JavaScript]({{ site.baseurl }}/chapters/13-appendix-js-basics.html)
1. [Bags of marbles]({{ site.baseurl }}/chapters/15-bags.html)
2. [Hierarchical models of categorization]()



