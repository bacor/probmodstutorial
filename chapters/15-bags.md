---
layout: chapter
title: Bags of marbles
description: Representing working models with probabilistic programs.
---



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
Suppose that we are now given three bags and also suppose we know how the marbles marbles of each color they contain:

~~~
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var bag1 = Categorical({vs: colors, ps: [.5,  .3,  .1, .05, .05]})
var bag2 = Categorical({vs: colors, ps: [.05, .25, .4, .25, .05]})
var bag3 = Categorical({vs: colors, ps: [.0,  .1,  .1, .3,  .5]})
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
This is uncertainty is even larger after observing a blue marble, since bag 1 and 2 contain nearly equal numbers of blue marbles.

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
Which bag do you think it came from, if you take into account that bag 2 is now twice as likely as bag 2 or 3?
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
var sampleMarble = function() {
  var bagNo = categorical({ vs: [1,2,3], ps: priorProbs })
  var bag = bags[bagNo-1]
  var marble = sample(bag);
  return marble
}

// Sample many marbles and plot their distribution
viz(repeat(5000, sampleMarble))
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

You draw from bag 1 or 2, untill you draw a marble that is neither red nor green. 
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

A categorical distribution was characterized by a simple parameter vector $$
\theta$$ indicating the probability of every category. 
A definition of the Dirichlet distribution is more involved and omitted here. 
We do need to know that a Dirichlet distribution is often parametrized by a concentration parameter $$\alpha$$ and a vector $$\beta$$ of the same length as the number of colors. 
Different choices of these parameters correspond to different ways of positioning the magnets if you like.
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

Try increasing the $$\alpha$$ from 0.1 to 1 and further. Explain what effect this parameter has. (Note that the limits of the plot are automatically adjusted!)
{: .homework }

Uncomment the second `beta` and (sytematically) play around with its values. (Tip: fix $$\alpha=1$$.) Can you explain what this parameter roughly does? Why is the first $$\beta$$ called a *symmetrical* Dirichlet distribution? 
{: .homework }

# A hierarchical model of a bag of marbles

Using a Dirichlet prior amounts to fixing a certain positioning of magnets behind a triangular darts board. Then you throw an arrow at the board (i.e., sample from the Dirichlet) and interpret the landing point as a categorical distribution over three colors. That is your bag. 

Let's implement that: draw the parameters $$\theta$$ of the categorical distribution from a Dirichlet distribution like the one on the left. The first starting 


~~~
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var alpha = 0.1
var beta = Vector([1,1,1])
var prior = Dirichlet(beta.mul(alpha))
var drawMarble = function() {
  var bag = Categorical({ vs: colors, ps: sample(prior) })
  return sample(bag)
}
viz(repeat(10, drawMarble))
~~~

asdfadf

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


# Full model
~~~~
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var bagToPrototype = mem(function(bag){return T.toScalars(dirichlet(ones([colors.length, 1])))})
var drawMarbles = function(bag, numDraws){
  var probs = bagToPrototype(bag);
  return repeat(numDraws, function(){return categorical({vs: colors, ps: probs})});
}

viz(drawMarbles('bag', 100))
viz(drawMarbles('bag', 100))
viz(drawMarbles('bag', 100))
viz(drawMarbles('other_bag', 100))
~~~~

Consider a simplification of this situation. There are several bags filled with marbles in five different colors. Each of these bags has a certain "prototypical" mixture of colors: one might contain mostly blue and green ones, another exclusively red ones, yet another one equally many marbles of every color, etc. The generative process is simple: we randomly pick a bag and then draw marbles from it. 
