---
layout: post
title: Basic To Poly
---

### Overview

All of my week was mostly consumed by asking questions like "Should this throw?" and "What are it's generators?". My work for this week was to make a mechanism for converting arbitrary expression into polynomials. Turns out, it isn't as trivial as it initially sounded. I will discuss my progress below. This week has been a lot more of logic involved, and thinking about how to go ahead instead of just development, so I do not have a lot to write about.

### The Idea

A multivariate polynomial is made of generators. A univariate polynomial is made of a single generator. Each term in a polynomial is a monomial which is multiplication of it's generators to a non-negative power, which also has a non-zero coefficient multiplied to it. Here are some polynomials and their generators :

```
x**2 + x**5          		->      x

sqrt(3) + sqrt(3)**3        ->		sqrt(3)

(1/x**2) + (1/x**5) 		-> 		(1/x)

x**2 + y**2 + x				-> 		x, y

2**x + 1					-> 		2**x
```

I hope you get the idea. The main task is given a `Basic`, we need to construct a polynomial with appropriate generator. The task is broken down into two parts. First off, we try and extract a generator from the given expression. Secondly, use the found (or user given) generator to construct the polynomial. 

Ideally the API should look as follows:

```
RCP<Basic> UIntPoly::from_basic(Basic x);
RCP<Basic> UIntPoly::from_basic(Basic x, Basic gen);
```

### My Progress

I have only worked on the univariate polynomial class as of now, and was to write the `from_basic` method just for the univariate classes for now. Although I just had to deal with the specific case of univariate integer polynomials, I haven't successfully been able to get the job done just yet.

The work started with converting `Symbol var` to `Basic var`. Initially, the generators of the polynomial class could only be symbols, however this needed to be changed, as we've seen generators can be anything! This change did not take a lot of effort.

I started off with a `find_gen` function which given a `Basic` will try and find a unique generator for the polynomial to be constructed. Ideally, if no generator/ more than one generator is found it will throw a runtime error. I covered most of the cases but a few still remain.

Also, an initial implementation of the `to_poly` functionality has been done, complete with the required API. Some cases related to conversion still need to be handled. I do not want to go into the logic involved into each conversion as it is tedious and not very structured. All the work done till now can be seen in [#998](https://github.com/symengine/symengine/pull/998).

I hope with all the information I have gained over this week, I will wind up the `Basic` to `UPoly` conversions for both the integer coefficients as well as the general expression coefficients by the end of next week. I also intend to make the logic more concrete, so that the same API can be used in the multivariate case.

### Miscellaneous Work

- I made some changes in the `expand` function related to polynomials. They mostly have to deal with removal of code, which has already been implemented. It is also a partial fix for [#886](https://github.com/symengine/symengine/issues/886), work is in [#999](https://github.com/symengine/symengine/pull/999).

Tata!