---
layout: post
title: "Briefly about Mutation Testing"
date: 2018-1-11
category: dev
tags: [testing, mutation testing]
header: "A few weeks ago <a href=\"/blog/2017/12/20/briefly-about-chaos-engineering\">I introduced chaos engineering</a> and this week I am going write briefly about mutation testing. Next week I'll compare these two concepts."
---
The goal of mutation testing or mutation analysis is to evaluate the quality of your existing software tests. In this kind of white-box testing your production code gets modified in tiny ways. A modified piece of code is called a mutant and if it makes a test fail, the mutant gets killed.

The more mutants are killed, the better your tests are.

But who should be a mutant created? There are well defined mutation operators that either mimic typical programming errors or force creation of valuable tests.

An example for the previous one can be using a wrong operator or variable name and for the latter one it can be forcing divisions by zero.

What does it mean if a mutant doesn't get killed? It means that either the modification never gets executed, so you have dead code, or that the tests do not cover the outputs enough.

The idea of mutation testing is based on two hypotheses. One is that most of the bugs are introduced by experienced programmers and those are small syntatic errors. The other hypothesis is that these small errors will cascade and couple to form other faults. Fault makes fault.

### The RIP model
To kill a mutant three conditions should be met which are collectively called the _RIP model_

* Reach the mutated statement
* Infect the program state with the test data
* Propagate the incorrect state to the output

Here is an example. This is the original code:

```
if a or b:
    c = 1
else:
    c = 0
```

And here is the modified code, where `or` is replaced by `and`:

```
if a and b:
    c = 1
else:
    c = 0
```

The _RIP_ is for strong mutation testing. If only the first two are satisfied, we can call it weak mutation testing. It's more related to code coverage.

If a mutant doesn't make a test fail so if it is not killed. it's called an equvalent mutant.

At the end of the test execution you will get your test suite's mutation score by dividing the number of mutants killed by the total number of mutants.

```
mutation score = number of mutants killed / total number of mutants
```

### Mutation operators

Just to give you some ideas how a program is modified by a mutation testing framework, I list a few so called mutation operators here.

* Statement deletion
* Statement duplication or insertion
* Remove Conditionals Mutator _(e.g. Replacement of boolean subexpressions with `true` and `false`)_
* Replacement of some arithmetic operations with others _(e.g. + with *, - with /)_
* Conditionals Boundary Mutator _(e.g. replace < with <=)_
* Negate Conditionals Mutator
* Invert Negatives Mutator
* Replacement of variables with others from the same scope (where variable types must be compatible)
* Return Values Mutator
* Remove method calls to void methods
* Non Void Method Call Mutator  _(i.e. replace return values by their type's defaults)_
* Constructor Call Mutator (i.e. constructor returns `null` instead of an instance)

Happy experimentation and testing!
