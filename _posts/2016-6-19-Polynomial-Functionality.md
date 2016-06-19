---
layout: post
title: Polynomial Functionalities
---

### Overview

The last week was spent on adding more functionality to all the three types of Integer Polynomials in SymEngine. Which meant that I had to basically wrap methods already existing in Piranha and Flint, and write new methods for the SymEngine polynomials to provide the required functionality. Details about said functions and the problems faced while implementing them are described in the rest of the post.

Also this Saturday, some friends and I decided to visit the hill station of Lonavala for the day. It was a replenishing experience, and a break from the regular routine. Here's us! 

![](http://srajangarg.github.io/assets/lonavala.jpg)

### Functions

I wanted to start off by adding a `div_upoly` functions for dividing two univariate polynomials. It is obvious that division of two polynomials with integer coefficients may very well result in polynomials with rational coefficients. But a rational polynomial class does not exist. So what does it mean to divide two univariate polynomials and get a integer polynomial quotient and remainder? The domain `ZZ[x]` is not a [Euclidean Domain](https://en.wikipedia.org/wiki/Euclidean_domain). Here is what we mean by Euclidean division,

```
Two polynomials p, q can uniquely be written as
p = quo * q + rem 
deg(rem) < deg(q) or rem = 0
```

Thus the euclidean division that we are so familiar with is not defined for this domain. So how do other libraries handle this division? I tried the division with Flint, Sage and SymPy.

```
In [14]: q = Poly(2*x)
In [15]: p = Poly(5*x**2)
In [16]: div(p, q, domain=ZZ)
```

Flint and Sage gave :

```
quo = Poly(2*x, x, domain='ZZ') 
rem = Poly(x**2, x, domain='ZZ')
```

while SymPy gave :

```
quo = Poly(0, x, domain='ZZ') 
rem = Poly(5*x**2, x, domain='ZZ')
```

This was interesting, as SymPy gave a different answer than the other two. I asked about this behaviour on gitter, and had a brief explanation given by [@jksuom](https://github.com/jksuom),

> The operation `quo_rem` is mathematically well defined only in so-called Euclidean domains. They are principal ideal domains (PIDs) equipped with a 'Euclidean function. `ZZ[x]` is not a PID, so there is no universal agreement on how `quo_rem` should be defined. I think that there are some appealing features in the way it is defined in sage, but one cannot say that SymPy's way would be wrong, either.

> It seems that the main (only?) practical use of `quo_rem` in non-Euclidean domains is for testing if an element is divisible by another. That happens when the remainder vanishes. Both ways of defining `quo_rem` in `ZZ[x]` can be used to decide this.

This explanation made a lot of sense, but I was anyways asked to open up [#11240](https://github.com/sympy/sympy/issues/11240), to know why SymPy handled division the way it did, and if it could be changed. Taking in all the inputs, I decided that I should not go ahead with a `div_upoly` function for now, instead I made a `divides_upoly` function which basically tells wether a polynomial divides the other or not. I had to implement the straight forward algorithm in SymEngine, while Flint already had a `divides` function and Piranha throws an exception on inexact division,  which I used to port it for Piranha polynomials.

Other methods which were wrapped were `gcd` and `lcm` for both the Flint and the Piranha polynomials. I have not been able to write these functions for SymEngine polynomials yet. The most trivial algorithms for finding GCD of two univariate polynomials depends on euclidean division, and that does not exist in the integer ring polynomials. Some more information from [@jksuom](https://github.com/jksuom), 

> The ring `ZZ[x]` is a unique factorization domain (UFD) even if it is not euclidean. It has `gcd` and `lcm`, but there is no euclidean algorithm for them.

Isuru suggested a hint for a simple algorithm, but I have not been able to follow it up yet. I'll catch up with it in the coming week. The next function I worked on was `pow`. Again this was already implemented in the two other libraries and I just had to wrap them. For SymEngine, I had to decide which algorithm to use, a naive repeated approach or the [exponention by squaring](https://en.wikipedia.org/wiki/Exponentiation_by_squaring) algorithm. I also benchmarked the `pow` methods of Piranha too. All the work and the benchmarks are in [#989](https://github.com/symengine/symengine/pull/989).

### Miscellaneous Work

Other functionalities like `multieval`, `derivative` and `get_lc` were also added for all the three intger polynomials.

There was an error in the Flint documentation, which was related with incorrect ordering of the arguments being passed to the `divides` function. It was reported in [#267](https://github.com/wbhart/flint2/issues/267).

Piranha required some implementations to be explicitly provided on the coefficient class, we are going to use. Thus, I had to overwrite some implementations like `gcd_impl`, `pow_impl` and `divexact_impl` for `SymEngine::intger_class` to work with `piranha::polynomial`.

I will write `gcd` and `lcm` for SymEngine polynomials, and start on `Basic -> Poly` conversion to add to the coercion framework decided for SymEngine, soon.

See you!