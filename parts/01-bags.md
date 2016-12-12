---
layout: chapter
tutorial: 1
title: Part 1 — Bags of marbles
description: Representing working models with probabilistic programs.
---

*The first section is adapted from [ProbMods, chapter 9](https://probmods.org/v2/chapters/09-hierarchical-models.html)*


# Hierarchical models of category learning

Human knowledge is organized hierarchically into levels of abstraction.  For instance, the most common or *basic-level* categories  (e.g. *dog*, *car*) can be thought of as abstractions across individuals, or more often across subordinate categories (e.g., *poodle*, *Dalmatian*, *Labrador*, and so on).  Multiple basic-level categories in turn can be organized under superordinate categories: e.g., *dog*, *cat*, *horse* are all *animals*; *car*, *truck*, *bus* are all *vehicles*. 

Some of the deepest questions of cognitive development are: How does abstract knowledge influence learning of specific knowledge?  How can abstract knowledge be learned? In this section we will see how such hierarchical knowledge can be modeled with *hierarchical generative models*: generative models with uncertainty at several levels, where lower levels depend on choices at higher levels.

Hierarchical models allow us to capture the shared latent structure underlying observations of multiple related concepts, processes, or systems -- to abstract out the elements in common to the different sub-concepts, and to filter away uninteresting or irrelevant differences. Perhaps the most familiar example of this problem occurs in learning about categories.  Consider a child learning about a basic-level kind, such as *dog* or *car*.  Each of these kinds has a prototype or set of characteristic features, and our question here is simply how that prototype is acquired.

The task is challenging because real-world categories are not homogeneous.  A basic-level category like *dog* or
*car* actually spans many different subtypes: e.g., *poodle*, *Dalmatian*, *Labrador*, and such, or
*sedan*, *coupe*, *convertible*, *wagon*, and so on.  The child observes examples of these sub-kinds or *subordinate*-level categories: a few poodles, one Dalmatian, three Labradors, etc. From this data she must infer what it means to be a dog in general, in addition to what each of these different kinds of dog is like.  Knowledge about the prototype level includes understanding what it means to be a prototypical dog and what it means to be non-prototypical, but still a dog. This will involve understanding that dogs come in different breeds which share features between them, but also differ systematically as well.

The ideas we will use to model this are best explained in a simpler setting. We return to its interpretation in part 2.

# Bags of marbles

Suppose you have a bag filled with marbles in five different colors `var colors = ['black', 'blue', 'green', 'orange', 'red']`. Some colors occur more frequently in the bag than others. 
If you randomly pick one of the marbles, the probability that the marble has a certain color is different for different colors. 
If 30% of the marbles in the bag are red for example, the probability of drawing a red marble is 30% or 0.3.

Since we are only interested in the probabilities of drawing marbles with a certain color, all we need to know about a bag is the percentage of marbles of every color. 
If we measure probabilities in numbers between 0 and 1 (instead of percentages), a bag with mostly black and green marbles and some others, be described by the list `var colorProbs = [0.35, 0.1, 0.35, .1, .1]`. 
This program picks marbles from such an urn (ignore the last line for now):

~~~
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var colorProbs = [0.35, 0.1, 0.35, .1, .1]
categorical({ vs: colors, ps: colorProbs})
~~~

Repeating this many times, we can approximate the *distribution* of colors in the bag by plotting the relative frequency of every color

~~~
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var colorProbs = [0.35, 0.1, 0.35, .1, .1]
var drawMarble = function() { categorical({ vs: colors, ps: colorProbs}) }
viz(repeat(100, drawMarble))
~~~

As you can see, it is only an approximation, since the frequency of black and green marbles is probably not *exactly* equal. 
The actual distribution is called a *categorical distribution*; it assign a probability to each of several categories --- colors, in our case:

$$
  p({\large\blacksquare}) = 0.35,
  \quad p({\large\color{blue}\blacksquare}) = 0.1,
  \quad p({\large\color{green}\blacksquare}) = 0.35,
  \quad p({\large\color{orange}\blacksquare}) = 0.1,
  \quad p({\large\color{red}\blacksquare}) = 0.1
$$

Together these probabilities form the parameters of the categorical distribution, and it is often convenient to group them in one list $$\theta = (0.35, 0.1, 0.35, 0.1, 0.1)$$.
In WebPPL you can define a categorical distribution by specifying its values `vs` (i.e., the colors) and the probabilities `ps` of each of those values. 
Here's a bag with a different composition of marbles and it's exact distribution:

~~~
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var colorProbs = [0.1, .3, 0.2, .3, .1]

var bag = Categorical({ vs: colors, ps: colorProbs})
viz(bag) // The actual distribution of colors

var drawMarble = function() { sample(bag) }
viz(repeat(100, drawMarble)) // The frequencies in 100 draws
~~~

A categorical distribution over `colors` thus serves as a model for a bag with marbles in five different colors.

In the two previous examples, the numbers in `colorProbs` summed up to 1. Why should that be the case?
{: .question }

You remove half of the `green` mables and replace them by `pink` marbles (a new color). Can you change the last block of code to visualize the new distribution?
{: .question }

# Multiple bags
Suppose that we are now given three bags and suppose we also know how many marbles of each color they contain:

~~~
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var bag1 = Categorical({vs: colors, ps: [.5,  .3,  .1, .05, .05]})
var bag2 = Categorical({vs: colors, ps: [.05, .25, .4, .25, .05]})
var bag3 = Categorical({vs: colors, ps: [.0,  .1,  .1, .3,  .5 ]})
viz(bag1); viz(bag2); viz(bag3);
~~~

As before, each bag is modelled by a categorical distribution.
But the parameters $$\theta_1, \theta_2$$ and $$\theta_3$$ of the three distributions are different. 
(What are they?)
The probability of drawing a blue ball from bag 1 for example, is 0.3, but 0.25 for bag 2 and 0.1 for bag 3.
<!-- The probability of drawing a color is only known *given* that we also know the bag we are drawing from. 
The probabilities are, in other words, *conditional probabilities*. -->
Once we know from which bag we are drawing, we also know the probability of drawing marbles of certain colors.
That is a *conditional* statement: we know the probability of drawing a color $$c$$, *given that* we know the bag $$b$$ being drawn from. 
This probability can be written as $$p(\text{color} \mid \text{bag})$$, where the pipe "$$\mid$$" indicates the conditioning and is read as "given".
As an example, here are some of the conditional probabilities from the last example:

$$
  p({\large\color{blue}\blacksquare} \mid \text{bag}_1) = 0.3,
  \quad p({\large\color{blue}\blacksquare} \mid \text{bag}_2) = 0.25,
  \quad p({\large\color{blue}\blacksquare} \mid \text{bag}_3) = 0.1
$$

Why do these not sum to one?
{: .question}


Now suppose someone draws a marble from one of the bags, but you can't see which.
Can we figure out from which bag it came?
If every bag contains marbles of only one color, we certainly can. 
(Why?)
But what if the bags contain multiple colors? 
You can be fairly certain that a black marble was drawn from bag 1, but some uncertainty remains. 
It could also have been drawn from 2 or 3, even though that is less likely.
This uncertainty is even larger after observing a blue marble, since bag 1 and 2 contain nearly equal numbers of blue marbles.

Let's further complicate the example and suppose that bag 1 and 3 are positioned much further away than bag 2.
As a result, drawing from bag 2 becomes twice as likely as drawing from bag 1 or 3.
This introduces a new (categorial) distribution that assigns a probability to each bag:

$$
p(\text{bag}_1) = 0.25, \qquad p(\text{bag}_2) = 0.5, \qquad p(\text{bag}_3) = 0.25
$$

Can you change the following code to visualize the distribution?
{: .question }

~~~
//var bags = ['bag1', ...
//var bagProbs = [...
viz(Categorical({vs: bags, ps: bagProbs}))
~~~

Again you are shown a marble drawn from an unkown bag.
This time, it is blue. 
Which bag do you think it came from, if you take into account that bag 2 is now twice as likely as bag 1 or 3?
{: .question }

Already, it's not completely clear anymore which bag that should be, even though you might have a hunch about it. 
So how to answer this in a more principled way?
One way is by start noting that we need to know how probable each bag is, given that we have just drawn a blue marble.
That is, we need to compute:

$$
p(\text{bag}_1 \mid {\large\color{blue}\blacksquare}),
\quad p(\text{bag}_2 \mid {\large\color{blue}\blacksquare}),
\quad \text{and}
\quad p(\text{bag}_3 \mid {\large\color{blue}\blacksquare}).
$$

These are the so called *posterior* probabilities: the probabilities *after* observing a blue marble. 
The probabilities $$p(\text{bag}_1)$$, $$p(\text{bag}_2)$$ and $$p(\text{bag}_3)$$ are called the *prior* probabilities: the probabilities of each of the bags *before* observing any marbles.
Finally, probabilities such as $$p({\large\color{blue}\blacksquare} \mid \text{bag}_2)$$ are called the *likelihoods*, since they capture how likely an observation is given the bag it was drawn from.
The problem is this.
We know the prior probabilities and the likelihoods. 
How to compute the posterior probabilities? 
The answer is Bayes' rule.

# Bayes' rule
Before we introduce Bayes' rule, it might help to abstract away a little.
The goal of a probabilistic model is to explain (predict) the observed *data*.
In the example, the observed data are (the colors of) the marbles we draw. 
In general, the actual distribution of colors in each bag will be unknown. 
But we will have certain *hypotheses* about the distribution.
One hypothesis reads that all balls are red, another that all balls are either blue or orange.
The three bags in our example embody yet three other hypotheses about the distribution of the data (i.e., the marble colors).
The question aksed above was which of these hypotheses best explains the data: which bag was probably used given that we have drawn a blue marble?
You then try to reason about the *hypotheses* using the data.

Bayes rule tells you how to do that.
More precisely, it says that the posterior probability $$p(h \mid d)$$ of a hypothesis $$h$$ given that you observed data $$d$$ is proportional to the likelihood $$p(d \mid h)$$ of the data under the hypothesis, multiplied by the prior probability of the hypothesis $$p(h)$$:

$$
  p(h \mid d) \propto p(d \mid h) \cdot p(h).
$$ 

The proportionality ($$\propto$$) means that the right hand side differs by a constant factor from the left hand side. 
The factor is $$1 / p(d)$$; it is a normalizing constant ensuring that the posterior probabilities sum to 1. 
For completeness, Bayes' rule reads

$$
  p(h \mid d) = \frac{p(d \mid h) \cdot p(h)}{p(d)}.
$$ 

In many (simple) cases Bayes' rule allows us to exactly calculate the posterior probability.
In particular, it answers the question about the bag that generated the blue marble in a principled way.
But in more complicated cases, it might not be possible to calculate the posterior exactly.
Then one can only hope to find a good approximation.
The next section is about that.

# Inference in a generative model
If you think about our three bags for a while, you might notice that it actually embodies a very simple story of how the data (marbles) are generated.
It goes as follows.

> "One day, someone picked one of bag 1, 2 and 3 with probabilities 0.25, 0.5 and 0.25. 
> She then randomly pulled one of the marbles out of the bag. 
> That's the marble we are looking at."

Admittedly, it won't win us the Booker prize, but the observation turns out to be quite useful. 
First, our model of the marbles is really a *generative* model as it directly models the probabilistic processes that generate the data.
Second and more importantly, it therefore gives us a recipe for generating new data. 
A recipe that can directly be translated into WebPPL code:

~~~
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var bag1 = Categorical({vs: colors, ps: [.5,  .3,  .1, .05, .05]})
var bag2 = Categorical({vs: colors, ps: [.05, .25, .4, .25, .05]})
var bag3 = Categorical({vs: colors, ps: [.0,  .1,  .1, .3,  .5]})
var bags = [bag1, bag2, bag3]

var priorProbs = [0.25, 0.5, 0.25]

// Function that implements the generative story
var drawMarble = function() {
  var bagNo = categorical({ vs: [1,2,3], ps: priorProbs })
  var bag = bags[bagNo-1]
  var marble = sample(bag);
  return marble
}

// Sample many marbles and plot their distribution
viz(repeat(5000, drawMarble))
~~~

The distribution shows the so called *marginal* distribution $$p(\text{color})$$.
This distribution already takes into account the prior probabilities of each of the bags; it is not the same as the conditional distribution $$p(\text{color} \mid \text{bag})$$. (Can you see that in the plot?)

How do you have to change the prior probabilities `priorProbs` to make red the color that is most likely to be drawn? Try it!
{: .question }

Just as before, by sampling many datapoints (colors) from our distribution and plotting the relative frequency of each, we could approximate the distribution. 
This is pretty naive way of approximating a distribution and much more sophisticated sampling methods exist. 
The nice thing about a probabilistic programming language such as WebPPL is that these methods have been built into the language, in the function `Infer`.
This function takes another function, such as our sampler, and infers the distribution over the output of that function.
The result is a special distribution object that allows you to, plot the distribution, calculate the probabilities of certain events, etc:

~~~
///fold:
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var bag1 = Categorical({vs: colors, ps: [.5,  .3,  .1, .05, .05]})
var bag2 = Categorical({vs: colors, ps: [.05, .25, .4, .25, .05]})
var bag3 = Categorical({vs: colors, ps: [.0,  .1,  .1, .3,  .5]})
var bags = [bag1, bag2, bag3]
///
var priorProbs = [0.25, 0.5, 0.25]
var distr = Infer({method: 'enumerate'}, function() {
  var bagNo = categorical({ vs: [1,2,3], ps: priorProbs })
  var bag = bags[bagNo-1]
  var marble = sample(bag);
  return marble
})
viz(distr)
~~~

And we can do more. Importantly, we can interact with `Infer` from inside the function of which we are trying to infer the output distribution. For example, the statement `condition(cond)` essentially instructs `Infer` to only consider the samples for which the condition `cond` was true. This allows us to infer the posterior distribution if we change two things in the previous code block. First, we need to return `bagNo` rather than `marble`, since we are interested in the the distribution over bags. Second, we need to condition on the marble being blue: `condition(marble == blue)`. That is implemented here:

~~~
///fold:
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var bag1 = Categorical({vs: colors, ps: [.5,  .3,  .1, .05, .05]})
var bag2 = Categorical({vs: colors, ps: [.05, .25, .4, .25, .05]})
var bag3 = Categorical({vs: colors, ps: [.0,  .1,  .1, .3,  .5]})
var bags = [bag1, bag2, bag3]
///
var priorProbs = [0.25, 0.5, 0.25]
var distr = Infer({method: 'enumerate'}, function() {
  var bagNo = categorical({ vs: [1,2,3], ps: priorProbs })
  var bag = bags[bagNo-1]
  var marble = sample(bag);
  condition(marble == 'blue')
  return bagNo
})
viz(distr)
~~~

Finally, we can answer our earlier question about which bag most likely generated the blue marble --- and pretty much every other question you might immediately have about this model. 

Someone tells you that a red marble has been drawn. 
Which bag do you think it came from?  
To find out, change the condition `marble == blue` in an appropriate way.
{: .question }

Now you're told that the drawn marble is red or green. 
Which bag did it come from? (Hint: `||` means "or" in JavaScript.)
{: .question }

You draw from bag 1 or 2, until you draw a marble that is neither red nor green. 
Which color will it most likely have? 
Please hand in your code.
(Hint: `!=` means "not equal"; `&&` means "and"; you can use multiple `condition(...)` statements.) 
{: .homework }


# Dirichlet distributions
Good, we nearly have all the basic techniques we need to implement a Hierarchical Bayesian model. 
Return for a moment to the more general view where the different bags were the hypotheses we entertained about the distributions of different colors.
We only considered three hypotheses (three bags), but there are many, many more.
Indeed, every bag or hypothesis corresponds to a categorical distribution, which is fully determined by the probabilities it assigns to each of the five colors --- by its parameter vector $$\theta$$, that is.
So every possible choice of five real numbers makes for a different hypothesis,(as long as the numbers sum up to 1).
(That's a lot: there are as many hypotheses as there are real numbers!)
When there were only three hypotheses to consider, the prior was itself a categorical distribution over the three hypotheses.
But what kind of prior can we use if we want to consider all the uncountably many hypotheses?

One convenient choice is a Dirichlet distribution. Let's introduce it in a simpler model with three colors: red, yellow and blue. Consider a bag that contains 50% red, 25% yellow and 25% blue marbles. The corresponding categorical distribution has a parameter vector $$\theta = (0.5, 0.25, 0.25)$$ and here are four different vizualizations of that same distribution:

<img src='{{site.baseurl}}/assets/img/categorical-distributions.svg ' width='100%' />

The top three should be self-explanatory; of interest is the triangle at the bottom.
All its side have length 1 and as a result, any distribution over three colors corresponds to one unique point in the triangle. 
The black point corresponds to the distribution shown above. 
The corners correspond to distributions that put all probability mass on one color (all marbles have the same color).
The reason that every distribution over 3 colors coresponds to a point on this 2d triangle is that there are only two degrees of freedom: once $$p(\text{red})$$ and $$p(\text{yellow})$$ have been fixed, it follows that $$p(\text{blue}) = 1 - p(\text{red}) - p(\text{yellow})$$. 
It also means that we can represent the space of all possible hypotheses about bags with three colors with a simple triangle. 
The prior distribution, remember, was a distribution over hypotheses. 
So a prior over the space of all hypotheses is a distribution over this triangle. 
What could that be?

Suppose you are playing triangular darts and someone has hidden very strong magnets behind the triangular darts board. 
First, the magnet is hidden behind the center of the board so most of your darts land in the center (see below, left). 
Next, the magnets are shifted to the corners, resulting in most arrows landing in the corners (right figure):

<img src='{{site.baseurl}}/assets/img/dirichlet.svg ' width='100%' />

The colored bars indicate some typical distributions corresponding to points on the triangle. 
One thing that you should notice is that the distributions on the left tend to be pretty uniform: all color are more or less equally probable. 
On the right, however, the distributions are strongly nonuniform and put nearly all probability mass on a single color. 

A categorical distribution was characterized by a parameter vector $$
\theta$$ indicating the probability of every category. 
A definition of the Dirichlet distribution is [more involved](https://en.wikipedia.org/wiki/Dirichlet_distribution) and omitted here. 
We do need to know that a Dirichlet distribution is often parametrized by a concentration parameter $$\alpha$$ and a vector $$\beta$$ of the same length as the number of colors. 
(The parameter vector then becomes $$\alpha \cdot \beta$$.)
Different choices of these parameters correspond to different ways of positioning the magnets, if you like.
Vizualizing actual samples from a Dirichlet distribution is --- of course --- pretty straightforward in WebPPL:

~~~~
var alpha = 0.1
var beta = Vector([1,1,1]) // symmetrical
// var beta = Vector([1,3,1])

viz(Infer({method: 'MCMC', samples:500}, function() {
  var s = dirichlet(beta.mul(alpha))
  return {red: s['data'][0], yellow: s['data'][1]}
}))
~~~~

Try increasing the $$\alpha$$ from 0.1 to 1 and further. Explain what effect this parameter has on the distribution and what that means for the typical draws from the distribution. (Note that the limits of the plot are automatically adjusted!)
{: .homework }

Uncomment the second `beta` and (systematically) play around with its values. (Tip: fix $$\alpha=1$$.) Can you explain what this parameter roughly does? Why is the first $$\beta$$ called a *symmetrical* Dirichlet distribution? 
{: .homework }

# A hierarchical model of a bag of marbles

Intuitively, using a Dirichlet prior amounts to fixing a certain positioning of magnets behind a triangular darts board. (If there are $$n$$ colors this must be a $$n-1$$ dimensional darts board.) You then throw an arrow at the board (i.e., sample from the Dirichlet) and interpret the landing point as a categorical distribution over the colors. That is your bag. 

Let's implement that: draw the parameters $$\theta$$ of the categorical distribution from a Dirichlet distribution like the one on the left. The first starting 


~~~
var colors = ['black', 'blue', 'green'];
var alpha = 0.1
var beta = Vector([1,1,1])
var prior = Dirichlet({ alpha: beta.mul(alpha) })
var drawMarble = function() {
  var colorProbs = T.toScalars(sample(prior))
  var bag = Categorical({ vs: colors, ps: colorProbs })
  return sample(bag)
}
viz(repeat(20, drawMarble))
~~~

Repeat the above simulation `10000` instead of `20` times. Is this what you expect to see?
{: .question}

Maybe not: it's roughly a uniform distribution. This happens because every time we draw a marble, we draw it from a *new* bag. 
(And since we used a symmetric prior, we are expected to sample from a uniform bag: precisely what you see. 
Try using an asymmetric prior such as  $$\beta=(2,1,1)$$.) 
This is not what we want: we want to sample *one* or more bags from the prior and use those same bags afterwards. 
In functional programming this can be very elegantly done using so called *memoization*. 
If you call a *memoized function* for the first time, the output corresponding to the inputs is *memorized*. 
The next time you call the function with the same inputs, it directly returns the output, without executing the function again. 
WebPPL implements memoization using `mem` and we can use it to draw multiple bags from the prior *only once*:

~~~
// A function without and with memoization
var getBag = function(bagName) { T.toScalars(dirichlet(Vector([1,1,1]))) }
var memGetBag = mem(getBag)

print('Without memoization: different every time')
print(getBag('bag1')); print(getBag('bag1'));

print('With memoization: the same every time for the same inputs')
print(memGetBag('bag1')); print(memGetBag('bag1'));

print('With memoization: but other inputs')
print(memGetBag('bag2')); print(memGetBag('bag2'));
~~~

As you see, the input `bagName` only serves an administrative purpose: differentiating different bags. 
Good. 
Let's put all this together.

(We will start using a handy shorthand: `dirichlet(...)` is the same as `sample(Dirichlet({ alpha: ... })`. More generally, `Distribution(...)` gives you a distribution object, while `distribution(...)` samples from it. )

~~~
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var alpha = 1
var beta  = Vector([1,1,1,1,1]) 

var getBag = mem(function(bagName) {
  var colorProbs = T.toScalars(dirichlet(beta.mul(alpha)))
  return Categorical({ vs: colors, ps: colorProbs })
})

var drawMarbles = function(bagName) {
  var bag = getBag(bagName);
  return repeat(1000, function() { sample(bag) });
}

viz(drawMarbles('bag')); viz(drawMarbles('bag'))
viz(drawMarbles('another_bag'))
~~~

As you can see, samples from the same bag follow the same distribution, but samples from different bags do not. 
Even though the distributions in different bags differ, they are similar in the sense that both distributions are fairly *flat*: the bags contain multiple colors.
In other words, they are not strongly *peaked*, i.e. they don't put all probability mass on a few colors.


What do you have to change in the model to get strongly peaked distributions? 
<!-- In other words, what do you have to change to ensure that all bags will strongly favour a few colors, even though different bags can still favour different colors?  -->
And how can you get really flat distributions in all bags? 
Explain what part of the code you changed, why that works and include plots of the two bags to illustrate your answer (you can download graphs by clicking on the 'tool' icon).
{: .homework }

In the previous exercise, you forced the model to show peaked distributions, but can you also force it to peak at a specific point? 
More precisely, can you ensure that all bags will contain mostly black marbles? 
What do you have to change and why? 
Again include plots to illustrate your answer.
{: .homework }

As you can see, changing the parameters of the Dirichlet prior has clear effects on the kind of bags you get.
But why should we decide on the value of these parameters, why not assume that the parameters $$\alpha$$ and $$\beta$$, too, are drawn from some distribution (a *hyperprior*)? 
As $$\beta$$ had to be a vector, you could for example draw it from a Dirichlet distribution (with certain parameters which you might fix --- or, if you get the hang of it, draw them from yet another distribution). 
And $$\alpha$$ can be drawn from many different distributions, such as an [exponential distribution](https://en.wikipedia.org/wiki/Exponential_distribution) with a parameter $$\lambda$$.
<!-- The distributions and you choose will depend on the phenomenon you are modelling. -->

This serves to illustrate how you can add more and more levels of abstraction to your model. 
Importantly, it is possible to learn something about these abstract levels from the data you have observed.
The next part of the tutorial is really about that.
But before we go there, let's see how another level of abstraction would be implemented.


~~~
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var lambda = .5,
    alpha  = exponential(lambda), 
    beta   = dirichlet(Vector([1,1,1,1,1]));

var getBag = mem(function(bagName) {
  var colorProbs = T.toScalars(dirichlet(beta.mul(5*alpha)));
  return Categorical({ vs: colors, ps: colorProbs });
})

// Vizualize the distributions of some bags
viz(getBag('bag1')); viz(getBag('bag2')); viz(getBag('bag3'))
~~~

A small value of $$\lambda$$ (e.g. 0.01) corresponds to a distribution that is likely to yield big numbers; a large value of $$\lambda$$ (e.g. 1) more likely yields smaller numbers (see below). Do you understand how changing $$\lambda$$ changes the distributions of the bags in the model above?
{: .question }

~~~
viz(Exponential({ a: .1 }))
viz(Exponential({ a: 10 }))
~~~

<hr />

Continue to [part 2 of the tutorial]({{ site.base_url }}/parts/02-hierarchical-models.html) or go to the [home page]({{ site.base_url }}).
**Warning:** once you navigate away, you loose all your code changes!
