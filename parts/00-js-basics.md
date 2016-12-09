---
layout: chapter
tutorial: 1
title: Part 0 — JavaScript basics
description: A very brief primer on JavaScript.
---

*This part is adapted from the [ProbMods' appendix](https://probmods.org/v2/chapters/13-appendix-js-basics.html)*


# Variables
The previous labs used the programming language R. In this tutorial, we will be using a language based on JavaScript. 
We quickly go through the basics of JavaScript and point out where it differs from R.

First things first.
In R you would define variables as `someVariable <- 3`; in JavaScript you can define variables using the keyword `var`:

~~~~
var someVariable = 3; // assign the value 3 to the variable someVariable
someVariable // when this is evaluated it looks up and returns the value 3
~~~~

<!-- (The `;` indicates that the line ends here; its use is optional.) -->
Note that you really have to use  `var`:

~~~
someVariable = 3
someVariable
~~~

Multiple variable can be assigned in the same line using a `,`.
To declare the end a line in JavaScript, use a `;`.
The use of `;` is optional, but can be useful for readability.

~~~~
var x = 3, y = 2; var z = x + y;
z
~~~~

# Data types in JavaScript
We have used many different types of data in R: numbers (`1`, `2.55392`, etc.) strings (`'hello'`), vectors (`c(1,2,3)`) and lists (`list(name='John')`). JavaScript has equivalents for most of these.

## Numbers, strings and booleans
Numbers, strings and booleans are roughly the same in R and JavaScript. You can do basic arithmetical operations:

~~~~
9/3 + (3 * 4)
~~~~

The `+` symbol is also used to concatenate strings:

~~~~
"My favorite food is " + "pizza"
~~~~

Numeric variables will automatically modified into strings during concatenation:

~~~~
3 + " is my favorite number"
~~~~

Boolean variables will be automatically changed into numbers when added (`false` becomes 0 and `true` becomes 1)

~~~~
true + true
~~~~

## Arrays
In R one often uses vectors, which can be construted using `c(1,2,3,4)`. JavaScript has no special vector objects, but instead allows you to use an *array* (list): a sequence of other values.

~~~~
["this", "is", "an", "array"]
~~~~

Arrays can be indexed using `[index]`.
*Note: indexing starts at 0*.

~~~~
var myArray = ["this", "is", "my", "array"]
myArray[1]
~~~~

The length can be computed using `.length`

~~~~
var myArray = ["this", "is", "my", "array"]
myArray.length
~~~~

A list of all the properties of arrays can be found [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/of).
Use caution with these.
Not all JavaScript methods are supported on arrays in WebPPL.
Some of these JavaScript methods will have their own WebPPL version.
A list of the WebPPL functions for arrays can be found [here](http://docs.webppl.org/en/master/functions/arrays.html).

## Objects
In real life, you encounter objects.
Objects have properties. 
In R we usually used lists to store objects: `bilbo <- list(firstName='Bilbo', lastName='Baggins')`. 
We then get the property `firstName` using `bilbo$firstName`.
In JavaScript you define objects using a different syntax and access its properties using `.property` or `["property"]`:

~~~~
var bilbo = { firstName: "Bilbo", lastName: "Baggins" }
print( bilbo.lastName )
print( bilbo["lastName"] )
~~~~


# Conditional statements

Just as in R, you can build complex expressions using the Boolean operators `||` (*or*) and `&&` (*and*):

~~~~
true && (true || false)
~~~~

Conditional `if`-`else` statements work the same as in R:

~~~~
// this line is a comment
if (1 == 2)  {     // the condition of "if"
  100              // the consequent ("then")
} else {
  (true || false)  // the alternative ("else")
}
~~~~

JavaScript has a very useful and common shorthand for `if` statements: it is called the "ternary" operator, using a question mark `?` and colon `:` to demarcate the three components.

The syntax is: `condition ? consequent : alternative`

~~~~
(1 == 2) ?    // the condition of "if"
  100 :       // the consequent ("then")
  (true || false)  // the alternative ("else")
~~~~

# Functions

Constructing functions in JavaScript is very similar to R:

~~~~
var double = function(x) {
  return x + x
}

double(3)
~~~~

There are a couple of differences. 
In R, you need to use brackets after `return`: `return(x + x)`, in JavaScript you don't.
In fact, in WebPPL, the use of the `return` keyword is optional (try removing it).
By default, WebPPL will return the last line of the function.
Still, we often use the `return` keyword for explicitness and clarity.

We can build procedures that manipulate any kind of value---even other procedures.
Here we define a function `twice` which takes a procedure and returns a new procedure that applies the original twice:

~~~~
var double = function(x) { return x + x }

var twice = function(f) {
  return function(x) {
    return f(f(x))
  }
}

var twiceDouble = twice(double)

twiceDouble(3)

// same as: twice(double)(3)
~~~~

When functions take other functions as arguments, that is called a higher-order function

## Higher-Order Functions

Higher-order functions can be used to represent common patterns of computation.
Several such higher-order functions are provided in WebPPL.

`map` is a higher-order function that takes a procedure and applies it to each element of a list.
For instance we could use map to test whether each element of a list of numbers is greater than zero:

~~~~
map(function(x){
  return x > 0
}, [1, -3, 2, 0])
~~~~

<!-- The `map` higher-order function can also be used to map a function of more than one argument over multiple lists, element by element.
For example, here is the MATLAB "dot-star" function (or ".*") written using `map2`, which maps over 2 lists at the same time:

~~~~
var dotStar = function(v1, v2){
  return map2(
    function(x,y){ return x * y },
    v1, v2)
}

dotStar([1,2,3], [4,5,6])
~~~~
 -->
`repeat` is a built-in function that takes another function as an argument. It repeats it how many ever times you want:

~~~~
var g = function(){ return 8 }
repeat(100, g)
~~~~

<hr />
Continue to [part 1 of the tutorial]({{ site.base_url }}/parts/01-bags.html) or go to the [home page]({{ site.base_url }}).

<!-- # Want to know more?
If you want to know more about Javascript, [JavaScript: The Good Parts](http://bdcampbell.net/javascript/book/javascript_the_good_parts.pdf) is an excellent introduction to the language.
Online tutorials can be found [here](http://www.w3schools.com/js/), [there](https://www.javascript.com), and [elsewhere](https://www.codeschool.com/learn/javascript). -->

<!-- Test your knowledge: [Exercises]({{site.baseurl}}/exercises/13-appendix-js-basics.html) -->
