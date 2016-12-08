---
layout: chapter
title: Bags of marbles
description: Representing working models with probabilistic programs.
#custom_js:
#- assets/js/box2d.js
#- assets/js/physics.js
#- assets/js/plinko.js
---

Human knowledge is organized hierarchically into levels of abstraction.  For instance, the most common or *basic-level* categories  (e.g. *dog*, *car*) can be thought of as abstractions across individuals, or more often across subordinate categories (e.g., *poodle*, *Dalmatian*, *Labrador*, and so on).  Multiple basic-level categories in turn can be organized under superordinate categories: e.g., *dog*, *cat*, *horse* are all *animals*; *car*, *truck*, *bus* are all *vehicles*. Some of the deepest questions of cognitive development are: How does abstract knowledge influence learning of specific knowledge?  How can abstract knowledge be learned? In this section we will see how such hierarchical knowledge can be modeled with *hierarchical generative models*: generative models with uncertainty at several levels, where lower levels depend on choices at higher levels.

Hierarchical models allow us to capture the shared latent structure underlying observations of multiple related concepts, processes, or systems -- to abstract out the elements in common to the different sub-concepts, and to filter away uninteresting or irrelevant differences. Perhaps the most familiar example of this problem occurs in learning about categories.  Consider a child learning about a basic-level kind, such as *dog* or *car*.  Each of these kinds has a prototype or set of characteristic features, and our question here is simply how that prototype is acquired.

The task is challenging because real-world categories are not homogeneous.  A basic-level category like *dog* or
*car* actually spans many different subtypes: e.g., *poodle*, *Dalmatian*, *Labrador*, and such, or
*sedan*, *coupe*, *convertible*, *wagon*, and so on.  The child observes examples of these sub-kinds or *subordinate*-level categories: a few poodles, one Dalmatian, three Labradors, etc. From this data she must infer what it means to be a dog in general, in addition to what each of these different kinds of dog is like.  Knowledge about the prototype level includes understanding what it means to be a prototypical dog and what it means to be non-prototypical, but still a dog. This will involve understanding that dogs come in different breeds which share features between them, but also differ systematically as well.

That observation can serve as a starting point for modelling categorization, when we think of categories as distributions over features. 
But before we get there, let's consider a much simpler scenario.

## A simple model: bag of marbles
Suppose you have a bag filled with marbles in five different colors `var colors = ['black', 'blue', 'green', 'orange', 'red']`. Some colors occur more frequently in the bag than others. 
If you randomly pick one of the marbles, the probability that the marble has a certain color is different for different colors. 
If 30% of the marbles in the bag are red for example, the probability of drawing a red marble is 30% or 0.3.
<!-- That is precisely the *fraction* of all the marbles that are red. -->

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

## Multiple bags
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

## Bayes' rule
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

## Inference in a generative model
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

Finally, we can answer our first question: which bag most likely generated the blue marble. And pretty much every other straightforward question you might have about this model. 

Someone tells you that a red marble has been drawn. Which bag do you think it came from?  To find out, change the condition `marble == blue` in an appropriate way.
{: .question }

Now you're told that the drawn marble is red or green. Which bag did it come from? (Hint: `||` means 'OR' in JavaScript)
{: .question }

## ...


The distribution shows the so called *marginal* probability of drawing a certain color, and no longer conditioned on the 
That is, this is no longer the conditional probability of drawing a color 

A principled answer to this question would try to compute the *probability of each of the bags*, given that you observed  taking into account how likely the bags are and that 

<!-- The uncertainty is clearer for blue balls since those are nearly as like to be drawn from bag 1 and bag 2. -->
The uncertainty we have about the bag the marble was drawn from, can be precisely quantified using probabilities. 

A simple experiment illustrates this. It works as follows: randomly pick one of the three bags, draw a marble and check whether it is blue.
If so, we write down the number of the bag and if not, we do nothing. Repeating that many times gives a list of the bags from which you have drawn a blue ball; something like $$(1, 1, 2, 1, 2, 2, 2, 3, 1, 1, 1, ...)$$
You would expect the bag that contains most blue balls to occur most frequently in that list. 
<!-- (Our implementation is slightly different: we write down the bag number if the marble is blue, and `false` otherwise. Later, we filter out all the `false`'s) -->

~~~
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var bag1 = Categorical({vs: colors, ps: [.5,  .3,  .1, .05, .05]})
var bag2 = Categorical({vs: colors, ps: [.05, .25, .4, .25, .05]})
var bag3 = Categorical({vs: colors, ps: [.0,  .1,  .1, .3,  .5]})
var bags = [bag1, bag2, bag3]

// In every experiment, draw a marble from a random urn 
// and return the bag number if marble was blue
var experiment = function() {
  var bagNo = uniformDraw([1,2,3])
  var bag = bags[bagNo-1]
  var marble = sample(bag);
  return (marble == 'blue') ? bagNo : false
}

// Repeat the experiment many times
var outcomes = repeat(10000, experiment)
print("The first 20 outcomes: " + filter(function(b){ b != false }, outcomes).slice(0,20))
print("The relative frequency of each bag:")
viz(filter(function(b){ b != false }, outcomes))
~~~

The graph shows which bag was responsible for what fraction of the blue marbles. 
Suppose you have just drawn a blue marble, but you don't know from which bag it came.
The probability that it came from each of the bags is indicated in the graph.
It thus shows the probability that you have drawn a marble from a certain bag, *given* that the marble is blue.

<!-- In other words, *given* that you have just drawn a blue marble, the graph shows for each of the bags, the probability that the marble came from that bag. -->

(Why) does it makes sense that this probability is highest for bag 1? 
{: .question }

Now consider a more complicated scenario where certain bags are more likely to have been used than others. 
Suppose for example that bag 1 and 3 are equally likely, while bag 2 is much more likely: two times as likely. 
(Perhaps bag 1 and 2 were placed further away.)
This initial distribution *over the bags* is called the *prior distribution*.

~~~
///fold:
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var bag1 = Categorical({vs: colors, ps: [.5,  .3,  .1, .05, .05]})
var bag2 = Categorical({vs: colors, ps: [.05, .25, .4, .25, .05]})
var bag3 = Categorical({vs: colors, ps: [.0,  .1,  .1, .3,  .5]})
var bags = [bag1, bag2, bag3]
///
// The prior distribution over [bag1, bag2, bag3]
var priorProbs = [0.25, 0.5, 0.25]

// Same experiment, but with a prior over the bags
var experiment = function() {
  var bagNo = categorical({ vs: [1,2,3], ps: priorProbs })
  var bag = bags[bagNo-1]
  var marble = sample(bag);
  return (marble == 'blue') ? bagNo : false
}

var outcomes = repeat(10000, experiment)
print("The first 20 outcomes: " + filter(function(b){ b != false }, outcomes).slice(0,20))
viz(filter(function(b){ b != false }, outcomes))
~~~

Why is it now much more likely that the marble came from bag 2?
Can you change the `priorProbs` in such a way that bag 3 becomes most likely? Or such that bag 1 *cannot* occur?
{: .question }

Let's think about this a bit more about this.
We started out by modelling three bags of marbles with categorical distributions, one for every bag.
Once we know from which bag we are drawing, we also know the probability of drawing marbles of certain colors.
That is a *conditional* statement: we know the probability of drawing a color $$c$$, *given that* we know the bag $$b$$ being drawn from. 
This probability can be abbreviated as 

$$
    p(\text{color} = c \mid \text{bag}=b)
$$

where the pipe "$$\mid$$" indicates the conditioning and is read as "given". Accordingly, this is a so called *conditional probability*. To make things concrete, in the last example, the probability of drawing a blue ball from bag 2 was  
$$p(\text{color} =\text{blue} \mid \text{bag}=2) = 0.25$$. Or similarly, $$p(\text{color} =\text{red} \mid \text{bag}=3) = 0.5$$. 

Having defined these distributions, we then performed an experiment that somehow gave us the probability that we had drawn from a certain bag, given that the marble was blue. That is, we somehow arrived at

$$
    p(\text{bag} = b \mid \text{color}=\text{blue}).
$$

The bag and the color have been 'flipped'.


## Bayes rule
The rule that justifies this 'flip' is [*Bayes Rule*](https://en.wikipedia.org/wiki/Bayes%27_theorem). 
To introduce Bayes rule, it might help to abstract away a little.
The goal of a probabilistic model is to explain (predict) the observed *data*.
In the example, the observed data were (the colors of) the marbles drawn from some bag. 
In general, the actual distribution of colors in that bag will be unknown. 
But we will have certain *hypotheses* about the distribution.
One hypothesis reads that all balls are red, another that all colors are equally probable.
The three bags in our example embody yet three other hypotheses about the distribution of the data (i.e., the marbles).
One question of interest could be which of these hypotheses best explains the data (e.g., "which bag was probably used given that we have drawn a blue marble?")
You then try to reason about the *hypotheses* using the data.

Bayes rule tells you how to do that.
More precisely, it says that the  probability $$p(h \mid d)$$ of a hypothesis $$h$$ given that you observed data $$d$$ is proportional to the likelihood $$p(d \mid h)$$ of the data under the hypothesis, multiplied by the prior probability of the hypothesis $$p(h)$$:

$$
  p(h \mid d) \propto p(d \mid h) \cdot p(h).
$$ 

From left to right, these probabilities are called the *posterior*, the *likelihood* and the *prior*. As you can see, the likelihood and the data play opposite roles in the posterior and the likelihood. That's exactly the 'flipping' we encountered before:

$$
  p(\text{bag} \mid \text{color}) \propto p(\text{color} \mid \text{bag}) \cdot p(\text{bag}).
$$ 

In the previous section, we *approximated* the distribution $$p(\text{bag} \mid \text{color})$$ by repeating a simple experiment many times; Bayes rule tells you how to do it exactly. 
The problem is that the posterior distribution $$p(h \mid d)$$ might be very complicated, in which case it approximating the distribution is the best you can do.
So how did we approximate the posterior?
The idea to keep in mind is simple: if you can generate many samples from a distribution, you can try to infer the distribution from the samples.





In practice, this *inference problem* is very difficult and many different algorithms exist to solve it.
The nice thing is that these algorithms have been built right into the *probabilistic* programming language WebPPL.
~~~
var getSample = function() {
  var A = flip()
  var B = flip()
  return A + B
}
var distribution = Infer({method: 'enumerate'}, getSample)
viz(distribution)
~~~


~~~
///fold:
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var bag1 = Categorical({vs: colors, ps: [.5,  .3,  .1, .05, .05]})
var bag2 = Categorical({vs: colors, ps: [.05, .25, .4, .25, .05]})
var bag3 = Categorical({vs: colors, ps: [.0,  .1,  .1, .3,  .5]})
var bags = [bag1, bag2, bag3]
///

var priorProbs = [0.25, 0.5, 0.25]
var model = Infer({method: "enumerate"}, function() {
    var bagNo = categorical({ vs: [1,2,3], ps: priorProbs })
    var bag = bags[bagNo-1]
    var marble = sample(bag);
    // condition(marble == 'blue')
    return marble
})
viz(model)
~~~

Change the code to return `bagNo` rather than `marble`. What do you see? 
{: .question }

The `condition(cond)` statement informs the `Infer` function to essentially ignore all samples where the `cond` is not `true`, i.e. where the condition is not satisfied. In that way you can infer conditional distributions.

While still returning `bagNo`, uncomment the line starting with `condition`. What do you see? 
{: .question }

Can you change the conditioning to get a vizualization of $$p(\text{bag} \mid \text{color} = \text{red})$$? And can you condition on the color being either red or blue? (Hint: `||` means 'OR' in JavaScript)
{: .question }

<!-- 

out with a model that gave us the probability of drawing a certain color *given* that we know if we are drawing from bag 1, bag 2 or bag 3. From that we reconstructed the probability of having drawn from a certain bag, *given* that the drawn marble was blue. In a way, we inverted our reasoning. This can be formally justified. Let's write 

$$
    p(\text{color} = d \mid \text{bag}=d)
    \qquad\text{or simply}\qquad p(c\mid b)
$$ 

for the probability that the color of the marble is $$c$$ (e.g. blue) *given that* the bag drawn from is bag $$b$$ (e.g. bag 1). 
This is called a *conditional probability* since it is a probability of drawing a marble with color $$c$$, *conditioned on* "$$\text{bag} = b$$" being the case. 
The pipe "$$\mid$$" indicates conditioning.
When "inverting" our reasoning above, we somehow got from

$$p(\text{color} = c \mid \text{bag}=b)
\qquad\text{to}\qquad
p(\text{bag}=b \mid \text{color} = c).
$$ 

The rule that allows you to get from left to right is the famous *Bayes rule*:

$$
p(b \mid c) = \frac{p(c \mid b) \cdot p(b)}{p(c)}.
$$ -->



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
