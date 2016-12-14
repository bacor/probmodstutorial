---
layout: chapter
tutorial: 1
title: Part 2 — Hierarchical models
description: The power of abstraction.
---

*This part is essentially a fragment of [ProbMods, chapter 9](https://probmods.org/v2/chapters/09-hierarchical-models.html).*


# From bags to prototypes
Part 1 started with the problem of learning abstract categories. How can a child learn a prototype, a set of characteristic features, for all the categories she encounters? This knowledge must moreover be hierarchical since a basic-level category like *dog* or *car* actually spans many different subtypes: e.g., *poodle*, *Dalmatian*, *Labrador*, and such, or
*sedan*, *coupe*, *convertible*, *wagon*, and so on. 
How can bags of marbles help making sense of this? The idea is that every bag has a certain "prototypical" mixture of colors that is reflected in the samples (marbles) you draw from it. So one might contain mostly blue and green ones, another exclusively red ones, yet another one equally many marbles of every color, etc. 




<!--
Hierarchical models allow us to capture the shared latent structure underlying observations of multiple related concepts, processes, or systems -- to abstract out the elements in common to the different sub-concepts, and to filter away uninteresting or irrelevant differences. Perhaps the most familiar example of this problem occurs in learning about categories.  Consider a child learning about a basic-level kind, such as *dog* or *car*.  Each of these kinds has a prototype or set of characteristic features, and our question here is simply how that prototype is acquired.

The task is challenging because real-world categories are not homogeneous.  A basic-level category like *dog* or
*car* actually spans many different subtypes: e.g., *poodle*, *Dalmatian*, *Labrador*, and such, or
*sedan*, *coupe*, *convertible*, *wagon*, and so on.  The child observes examples of these sub-kinds or *subordinate*-level categories: a few poodles, one Dalmatian, three Labradors, etc. From this data she must infer what it means to be a dog in general, in addition to what each of these different kinds of dog is like.  Knowledge about the prototype level includes understanding what it means to be a prototypical dog and what it means to be non-prototypical, but still a dog. This will involve understanding that dogs come in different breeds which share features between them, but also differ systematically as well.

## Bags of marbles
Consider a simplification of this situation. There are several bags filled with marbles in five different colors. Each of these bags has a certain "prototypical" mixture of colors: one might contain mostly blue and green ones, another exclusively red ones, yet another one equally many marbles of every color, etc. The generative process is simple: we randomly pick a bag and then draw marbles from it. 
 -->


<!-- We will draw marbles out of several different bags. There are five marble colors. Each bag has a certain "prototypical" mixture of colors. This generative process is represented in the following WebPPL example using the Dirichlet distribution (the Dirichlet is the higher-dimensional analogue of the Beta distribution).

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

As this examples shows, `mem` is particularly useful when writing hierarchical models because it allows us to associate arbitrary random draws with categories across entire runs of the program. In this case it allows us to associate a particular mixture of marble colors with each bag. The mixture is drawn once, and then remains the same thereafter for that bag. Intuitively, you can see how each sample is sufficient to learn a lot about what that bag is like; there is typically a fair amount of similarity between the empirical color distributions in each of the four samples from `bag`.  In contrast, you should see a different distribution of samples from `other_bag`.
 -->

<!-- Early in part 1, we considered three bags of which we knew the distributions. That allowed us to for example answer which bag most likely generated which color, but you can't do much more. Importantly, we could not infer the distribution of marbles in the bags in the light of some observed data. But once we assume that the distributions come from a another distribution, we can.  -->
We will now generate three different bags, and try to learn about their respective color prototypes by conditioning on observations. We represent the results of learning in terms of the [*posterior predictive* distribution](https://en.wikipedia.org/wiki/Posterior_predictive_distribution) for each bag. 
This distribution captures the probabilities of new observations (predictions, hence *predictive*) after you have already observed some data (*posterior*). 
It shows the new beliefs about the marbles in the different bags. 

Expanding on that a little, remember that in part 1 we tried to infer from which bag a blue marble was drawn. 
Suppose we now observe a blue and a red marble, drawn from a single bag and want to *predict* the color of the next marble (from the same bag). 
One could find the bag with highest posterior probability (here: bag 3) and use the marble distribution in that bag to make new preditions. 
But this would ignore the uncertainty we still have about the bag used.
In a Bayesian context, one would take that uncertainty into account by *averaging* over the bags according to their posterior probability.
That means that our predictions will be heavily influenced by bag 3, but we do not completely exclude that the marbles came from bag 1 or 2, as you can see here:

```
///fold:
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var bag1 = Categorical({vs: colors, ps: [.5,  .3,  .1, .05, .05]})
var bag2 = Categorical({vs: colors, ps: [.05, .25, .4, .25, .05]})
var bag3 = Categorical({vs: colors, ps: [.0,  .1,  .1, .3,  .5]})
var bags = [bag1, bag2, bag3]
///
var priorProbs = [0.25, 0.5, 0.25]
var posterior = Infer({method: 'enumerate'}, function() {
  var bagNo = categorical({ vs: [1,2,3], ps: priorProbs })
  var bag = bags[bagNo-1]
  observe(bag, 'blue'); observe(bag, 'red')
  return bagNo
})

var predictive = Infer({ method: 'enumerate' }, function() {
  var bagNo = sample(posterior)
  return sample(bags[bagNo-1])
})

print('posterior:'); viz(posterior)
print('predictive:'); viz(predictive)
print('bag3:'); viz(bag3)
```

(Earlier we used a `condition` statement rather than `observe`. In this case, `observe(model, outcome)` has the same effect as `condition(sample(model) == outcome)`.) 

In this example you see two strategies for making new predictions: predict colors according to their probability under the most likely hypothesis (i.e., bag 3), or according to the posterior predictive distribution.
Both methods yield different predictions.
Importantly, they differ in their predictions about black marbles. 
Why are the black marbles telling?
Which of the two methods would you consider to be more 'cautious'?
{: .question }

Note that we can compress the previous example by inferring the predictive distribution directly:

~~~
///fold:
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var bag1 = Categorical({vs: colors, ps: [.5,  .3,  .1, .05, .05]})
var bag2 = Categorical({vs: colors, ps: [.05, .25, .4, .25, .05]})
var bag3 = Categorical({vs: colors, ps: [.0,  .1,  .1, .3,  .5]})
var bags = [bag1, bag2, bag3]
///
var priorProbs = [0.25, 0.5, 0.25]
viz(Infer({method: 'enumerate'}, function() {
  var bagNo = categorical({ vs: [1,2,3], ps: priorProbs })
  var bag = bags[bagNo-1]
  observe(bag, 'blue'); observe(bag, 'red')
  return sample(bag)
}))
~~~

In short, we can use the predictive distribution to understand how the model predictions change in the light of observed data. 
As said, we will now consider that for observations from three different bags.
We will also draw a sample from the posterior predictive distribution on a new bag called `bagN`, for which we have had no observations.

~~~
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var observedData = [
{bag: 'bag1', draw: 'blue'}, {bag: 'bag1', draw: 'blue'},
{bag: 'bag1', draw: 'black'},{bag: 'bag1', draw: 'blue'},
{bag: 'bag1', draw: 'blue'}, {bag: 'bag1', draw: 'blue'},
{bag: 'bag2', draw: 'blue'}, {bag: 'bag2', draw: 'green'},
{bag: 'bag2', draw: 'blue'}, {bag: 'bag2', draw: 'blue'},
{bag: 'bag2', draw: 'blue'}, {bag: 'bag2', draw: 'red'},
{bag: 'bag3', draw: 'blue'}, {bag: 'bag3', draw: 'blue'},
{bag: 'bag3', draw: 'blue'}, {bag: 'bag3', draw: 'blue'},
{bag: 'bag3', draw: 'blue'}, {bag: 'bag3', draw: 'orange'}]

var predictives = Infer({method: 'MCMC', samples: 20000}, function(){

  var getBag = mem(function(bagName) {
    var colorProbs = T.toScalars(dirichlet(Vector([1,1,1,1,1])))
    return Categorical({ vs: colors, ps: colorProbs })
  })

  // Observe all data and return samples
  map(function(d) { observe(getBag(d.bag), d.draw) }, observedData)
  return {bag1: sample(getBag('bag1')), bag2: sample(getBag('bag2')),
          bag3: sample(getBag('bag3')), bagN: sample(getBag('bagN'))}
})
viz.marginals(predictives)
~~~

This generative model describes the prototype mixtures in each bag, but it does not attempt learn a common higher-order prototype. It is like learning separate prototypes for subordinate classes *poodle*, *Dalmatian*, and *Labrador*, without learning a prototype for the higher-level kind *dog*.  Specifically, inference suggests that each bag is predominantly blue, but with a fair amount of residual uncertainty about what other colors might be seen. There is no information shared across bags, and nothing significant is learned about `bagN` as it has no observations and no structure shared with the bags that have been observed.

Now let us introduce another level of abstraction: a global prototype that provides a prior on the specific prototype mixtures of each bag.

~~~~
///fold:
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var observedData = [
{bag: 'bag1', draw: 'blue'}, {bag: 'bag1', draw: 'blue'},
{bag: 'bag1', draw: 'black'},{bag: 'bag1', draw: 'blue'},
{bag: 'bag1', draw: 'blue'}, {bag: 'bag1', draw: 'blue'},
{bag: 'bag2', draw: 'blue'}, {bag: 'bag2', draw: 'green'},
{bag: 'bag2', draw: 'blue'}, {bag: 'bag2', draw: 'blue'},
{bag: 'bag2', draw: 'blue'}, {bag: 'bag2', draw: 'red'},
{bag: 'bag3', draw: 'blue'}, {bag: 'bag3', draw: 'blue'},
{bag: 'bag3', draw: 'blue'}, {bag: 'bag3', draw: 'blue'},
{bag: 'bag3', draw: 'blue'}, {bag: 'bag3', draw: 'orange'}]
///

var predictives = Infer({method: 'MCMC', samples: 20000}, function(){

  // Shared alpha and beta
  var alpha = 1;
  var beta = dirichlet(Vector([1,1,1,1,1]));
  
  var getBag = mem(function(bagName) {
    var colorProbs = T.toScalars(dirichlet(beta.mul(5*alpha)));
    return Categorical({ vs: colors, ps: colorProbs });
  })

  // Observe all data and return samples
  map(function(d) { observe(getBag(d.bag), d.draw) }, observedData)
  return {bag1: sample(getBag('bag1')), bag2: sample(getBag('bag2')),
          bag3: sample(getBag('bag3')), bagN: sample(getBag('bagN'))}
})
viz.marginals(predictives)
~~~~

Compared with inferences in the previous example, this extra level of abstraction enables faster learning: more confidence in what each bag is like based on the same observed sample.  This is because all of the observed samples suggest a common prototype structure, with most of its weight on `blue` and the rest of the weight spread uniformly among the remaining colors.  Statisticians sometimes refer to this phenomenon of inference in hierarchical models as "sharing of statistical strength": it is as if the sample we observe for each bag also provides a weaker indirect sample relevant to the other bags.  In machine learning and cognitive science this phenomenon is often called *learning to learn* or *transfer learning.* Intuitively, knowing something about bags in general allows the learner to transfer knowledge gained from draws from one bag to other bags.  This example is analogous to seeing several examples of different subtypes of dogs and learning what features are in common to the more abstract basic-level dog prototype, independent of the more idiosyncratic features of particular dog subtypes.


<!-- A particularly striking example of "sharing statistical strength" or "learning to learn" can be seen if we change the observed sample for bag 3 to have only two examples, one blue and one orange.  (Remove all but one `{bag: 'bag3', draw: 'blue'}` from `observedData` in program above.) In a situation where we have no shared higher-order prototype structure, inference for bag3 from these observations suggests that `blue` and `orange` are equally likely.  However, when we have inferred a shared higher-order prototype, then the inferences we make for bag 3 look much more like those we made before (with six observations: five blue, one orange), because the learned higher-order prototype tells us that blue is most likely to be highly represented in any bag regardless of which other colors (here, orange) may be seen with lower probability.
 -->
Learning about shared structure at a higher level of abstraction also supports inferences about new bags without observing *any* examples from that bag (such as `bagN` in the above examples): a hypothetical new bag could produce any color, but is likely to have more blue marbles than any other color. We can imagine hypothetical, previously unseen, new subtypes of dogs that share the basic features of dogs with more familiar kinds but may differ in some idiosyncratic ways.

A particularly striking example of "sharing statistical strength" or "learning to learn" can be seen if we change the observed sample for bag 3 to have only two examples, one blue and one orange. 
Remove all but one `{bag: 'bag3', draw: 'blue'}` from `observedData` in two programs above.
You should see a clear difference in the inference for bag 3 between the two programs. 
Can you explain this difference?
{: .homework }


# Learning Overhypotheses: Abstraction at the Superordinate Level

Hierarchical models also allow us to capture a more abstract and even more important "learning to learn" phenomenon, sometimes called learning *overhypotheses*.  Consider how a child learns about living creatures (an example we adapt from the psychologists Liz Shipley and Rob Goldstone).   We learn about specific kinds of animals -- dogs, cats, horses, and more exotic creatures like elephants, ants, spiders, sparrows, eagles, dolphins, goldfish, snakes, worms, centipedes  -- from examples of each kind.  These examples tell us what each kind is like: Dogs bark, have four legs, a tail.  Cats meow, have four legs and a tail.  Horses neigh, have four legs and a tail. Ants make no sound, have six legs, no tail.   Robins and eagles both have two legs, wings, and a tail; robins sing while eagles cry.  Dolphins have fins, a tail, and no legs; likewise for goldfish.  Centipedes have a hundred legs, no tail and make no sound.  And so on.  Each of these generalizations or prototypes may be inferred from seeing several examples of the species.

But we also learn about what kinds of creatures are like *in general*.  It seems that certain kinds of properties of animals are characteristic of a particular kind: either every individual of a kind has this property, or none of them have it.  Characteristic properties include number of legs, having a tail or not, and making some kind of sound.  If one individual in a species has four legs, or six or two or eight or a hundred legs, essentially all individuals in that species have that same number of legs (barring injury, birth defect or some other catastrophe).  Other kinds of properties don't pattern in such a characteristic way.  Consider external color.  Some kinds of animals are homogeneous in coloration, such as dolphins, elephants, sparrows.  Others are quite heterogeneous in coloration: dogs, cats, goldfish, snakes. Still others are intermediate, with one or a few typical color patterns: horses, ants, eagles, worms.

This abstract knowledge about what animal kinds are like can be extremely useful in learning about new kinds of animals. Just one example of a new kind may suffice to infer the prototype or characteristic features of that kind: seeing a spider for the first time, and observing that it has eight legs, no tail and makes no sound, it is a good bet that other spiders will also have eight legs, no tail and make no sound.  The specific coloration of the spider, however, is not necessarily going to generalize to other spiders.  Although a basic statistics class might tell you that only by seeing many instances of a kind can we learn with confidence what features are constant or variable across that kind, both intuitively and empirically in children's cognitive development it seems that this "one-shot learning" is more the norm. How can this work?  Hierarchical models show us how to formalize the abstract knowledge that enables one-shot learning, and the means by which that abstract knowledge is itself acquired [@Kemp2007].

We can study a simple version of this phenomenon by modifying our bags of marbles example, articulating more structure to the hierarchical model as follows.  We now have two higher-level parameters: `beta` describes the expected proportions of marble colors across bags of marbles, while `alpha`, a real number, describes the strength of the learned prior -- how strongly we expect any newly encountered bag to conform to the distribution for the population prototype `beta`.  For instance, suppose that we observe that `bag1` consists of all blue marbles, `bag2` consists of all green marbles, `bag3` all red, and so on. This doesn't tell us to expect a particular color in future bags, but it does suggest that bags are very regular---that all bags consist of marbles of only one color.

~~~~
var colors = ['black', 'blue', 'green', 'orange', 'red'];
var observedData = [
{bag: 'bag1', draw: 'blue'}, {bag: 'bag1', draw: 'blue'}, {bag: 'bag1', draw: 'blue'},
{bag: 'bag1', draw: 'blue'}, {bag: 'bag1', draw: 'blue'}, {bag: 'bag1', draw: 'blue'},
{bag: 'bag2', draw: 'green'}, {bag: 'bag2', draw: 'green'}, {bag: 'bag2', draw: 'green'},
{bag: 'bag2', draw: 'green'}, {bag: 'bag2', draw: 'green'}, {bag: 'bag2', draw: 'green'},
{bag: 'bag3', draw: 'red'}, {bag: 'bag3', draw: 'red'}, {bag: 'bag3', draw: 'red'},
{bag: 'bag3', draw: 'red'}, {bag: 'bag3', draw: 'red'}, {bag: 'bag3', draw: 'red'},
{bag: 'bag4', draw: 'orange'}]

var predictives = Infer({method: 'MCMC', samples: 30000}, function(){

  var alpha = gamma(2,2)
  var beta = dirichlet(ones([5, 1]))
  
  var getBag = mem(function(bagName) {
    var colorProbs = T.toScalars(dirichlet(beta.mul(alpha)));
    return Categorical({ vs: colors, ps: colorProbs })
  })

  map(function(d) { observe(getBag(d.bag), d.draw) }, observedData)
  return {bag1: sample(getBag('bag1')), bag2: sample(getBag('bag2')),
          bag3: sample(getBag('bag3')), bag4: sample(getBag('bag4')),
          bagN: sample(getBag('bagN')),
          alpha: alpha}
});
viz.marginals(predictives)
~~~~

Consider the fourth bag, for which only one marble has been observed (orange): we see a very strong posterior predictive distribution focused on orange -- a "one-shot" generalization.  This posterior is much stronger than the single observation for that bag can justify on its own.  Instead, it reflects the learned overhypothesis that bags tend to be uniform in color.

To see that this is real one-shot learning, contrast with the predictive distribution for a new bag with no observations: `bagN` gives a mostly flat distribution. Little has been learned in the hierarchical model about the specific colors represented in the overall population; rather we have learned the abstract property that bags of marbles tend to be uniform in color. Hence, a single observation from a new bag is enough to make strong predictions about that bag even though little could be said prior to seeing the first observation.

We have also generated the posterior distribution on `alpha`, representing how strongly the prototype distribution captured in `beta`, constrains each individual bag -- how much each individual bag is expected to look like the prototype of the population. You should see that the inferred values of `alpha` are typically significantly less than 1.  This means roughly that the learned prototype in `beta` should exert less influence on prototype estimation for a new bag than a single observation.  Hence the first observation we make for a new bag mostly determines a strong inference about what that bag is like.

Now change the `observedData` in the above model as follows:
~~~~norun
var observedData = [
{bag: 'bag1', draw: 'blue'}, {bag: 'bag1', draw: 'red'}, {bag: 'bag1', draw: 'green'},
{bag: 'bag1', draw: 'black'}, {bag: 'bag1', draw: 'red'}, {bag: 'bag1', draw: 'blue'},
{bag: 'bag2', draw: 'green'}, {bag: 'bag2', draw: 'red'}, {bag: 'bag2', draw: 'black'},
{bag: 'bag2', draw: 'black'}, {bag: 'bag2', draw: 'blue'}, {bag: 'bag2', draw: 'green'},
{bag: 'bag3', draw: 'red'}, {bag: 'bag3', draw: 'green'}, {bag: 'bag3', draw: 'blue'},
{bag: 'bag3', draw: 'blue'}, {bag: 'bag3', draw: 'black'}, {bag: 'bag3', draw: 'green'},
{bag: 'bag4', draw: 'orange'}]
~~~~

What kind of overhypothesis is learned with the new data? Explain how this can be seen from $$\alpha$$ and the inference for bag 4.
{: .homework }


<!-- Intuitively, the observations for bags one, two and three should now suggest a very different overhypothesis: that marble color, instead of being homogeneous within bags but variable across bags, is instead variable within bags to about the same degree that it varies in the population as a whole.  We can see this inference represented via two coupled effects.  First, the inferred value of `alpha` is now significantly *greater* than 1, asserting that the population distribution as a whole, `phi`, now exerts a strong constraint on what any individual bag looks like.  Second, for a new `'bag4` which has been observed only once, with a single orange marble, that draw is now no longer very influential on the color distribution we expect to see from that bag; the broad distribution in `phi` exerts a much stronger influence than the single observation. -->



# Thoughts on Hierarchical Models

We have just seen several examples of *hierarchical Bayesian models*: generative models in which there are several levels of latent random choices that affect the observed data. In particular a hierarchical model is usually one in which there is a branching structure in the dependence diagram, such that the "deepest" choices affect all the data, but they only do so through a set of more shallow choices which each affect some of the data, and so on.

Hierarchical model structures give rise to a number of important learning phenomena: transfer learning (or learning-to-learn), the blessing of abstraction, and learning curves with fairly abrupt transitions. This makes them important for understanding human learning, as well as useful for creating artificial intelligence that makes the best use of available data.

