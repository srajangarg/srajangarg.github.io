---
layout: post
title: Finding Gnererators & Polynomial Conversions
---

### Overview

This week I continued my work from last week, but in a cleaner and structured fashion. The PR with last week's work was closed in favour of a new PR, with better and scalable code. I'll briefly explain how the functionality is implemented and the main ideas behind it. All the work is in [#1003](https://github.com/symengine/symengine/pull/1003).

### Finding Generators

You can refer to last week's post to know what a generator for a polynomial is at a preliminary level. An expression may have any number of generators, but our job is to get the least number of generators which will be able to construct the given expression. The only rule is that a generator cannot be a `Number` (ie `Rational` or `Integer`). So `sqrt(3)` and `pi` can be generators but `2` or `1/2` can't.

Initially, the approach I was trying out was not general (as I catered to only the Univariate Int case), and expanding it to other polynomials would prove to be immensely difficult. Isuru suggested that the funciton should return all the possible generators. This will be the most general case for this function, and we can adapt to specific cases based on how many / which generators the function returns. I will try and summarize how the function works at a high level.

If the expression passed to `find_gen` is a

* `Add` and `Mul` : We can just call `find_gen` recursively on each of the expressions being added/multiplied. `x + y` or `x*y` will return both `x` and `y` as the generators. This follows from the fact that polynomials can be multiplied or added up to give the resultant polynomial.

* `Number` : Do nothing, as numbers can never act as generators of polynomials.

* `Pow` : Few cases arise in if the expression is a `Pow`. If the exponent is a positive integer, it suffices to find the generators of only the base of the expression. If it is a negative integer, we update the current generator set with `base**(-1)`. For eg.

```
find_gen((x + y)**4) = find_gen(x + y) = (x, y)
find_gen((x**-2)) = (x**-1)
```

If the exponent is not an integer, the situation becomes a little complicated. It would seem intuitive that the generators in this case would be the base powered to the generators of the exponent. Like so,

```
find_gen(base**exp) = base**find_gen(exp)
eg. 
find_gen(2**(x+y)) = 2**find_gen(x+y) = (2**x, 2**y)
```

This would seem to be working, but actually the same `find_gen` function cannot be used to get the generators of the exponent expression. The `find_gen` works in a different way once we are "inside" an exponent. Take for example :

```
// Incorrect
find_gen(2**(x + (1/2)))
= 2**find_gen(x + (1/2))
= 2**x

// Correct
find_gen(2**(x + (1/2)))
= 2**find_gen_pow(x + (1/2))
= (2**x, 2**(1/2))
```

`find_gen_pow` is another function which returns generators keeping in mind that the current expression it is dealing with is actually the exponent of another expression. So, it's behaviour varies from the simple `find_gen`. Here is another example : 

```
// Incorrect
find_gen(2**(x**2 + 3*x + (1/2))
= 2**find_gen(x**2 + 3*x + (1/2))
= 2**x

// Correct
find_gen(2**(x**2 + 3*x + (1/2))
= 2**find_gen_pow((x**2 + 3*x + (1/2))
= (2**x, 2**(x**2), 2**(1/2))
```

* `Basic` : For all other structures like `Symbol` and `Function`, they themselves act as generators of the polynomial. We just need to update the generator set.

It is to be kept in mind that whenever we obtain a new potential generator, we update the current generator set. This may lead to modification of an already existing generator or add in a new one. This method thus takes care of some cases, not done by SymPy. A similar updation rule is followed for `find_gen_pow` where this is done on the coefficients of each expression instead of their powers. 

```
// "find_gen"
gen_set = (x**(1/2))

// addition
update_gen_set(z)
gen_set = (x**(1/2), z)

// modification
update_gen_set(x**(1/3))
gen_set = (x**(1/6))

// no change
update_gen_set(x)
gen_set = (x**(1/2))
```

```
// "find_gen_pow"
gen_set = (x/2)

// addition
update_gen_set(x**(1/2))
gen_set = (x/2, x**(1/2))

// modification
update_gen_set(x/3)
gen_set = (x/6)

// no change
update_gen_set(x)
gen_set = (x/2)
```

I have a short description on the storage of these generators in the PR description and subsequent comments. Please do have a look!

### Polynomial Conversions

The next half of the week's work involved actually converting a `Basic` into a polynomial. The API is too look as follows

```
RCP<Basic> UIntPoly::from_basic(Basic x);
RCP<Basic> UIntPoly::from_basic(Basic x, Basic gen);
```

The user can either supply a generator or one will automatically be deduced and used. The functions will throw of exactly one generator is not found, as we are dealing with univariate polynomials. Initially, I planned to write two different functions for `UIntPoly` and `UExprPoly` conversions, but soon I realized that most of the code was going to be identical. So, I formed a template function base with most of the common code, while the specific visitors inherit from this template base class with special methods.

The actual conversion was not too difficult, it just had to be broken down into cases like the `find_gen` problem. I leave it to the reader to try and theorize how this can be done. For eg. Imagine an `Add`, you can call `b2poly` on each expression added and add up the resulting polynomials. The code right now is scalable, and if we add in more polynomial classes like `URatPoly` there will be no issues in adding them in.

### Miscellaneous Work

- Templatized `pow_upoly` so that it can be used by any general polynomial class. Also removed some redundant code in the `ExpandVisitor` related to said function. Can be seen in [#1010](https://github.com/symengine/symengine/pull/1010)

- I was just testing out how the parser is working out with the basic to polynomial conversions. It is working very seamlessly, constructing polynomials has never been easier :smile:

```
s = "2*(x+1)**10 + 3*(x+2)**5";
poly = UIntPoly::from_basic(parse(s));

s = "((x+1)**5)*(x+2)*(2*x + 1)**3";
poly = UIntPoly::from_basic(parse(s));
```

Laters!