---
layout: post
title: Multivariate Polynomials
---

### Overview

After college has started, work has been slower than usual. Last week I finally wound up Rational Polynomials in [#1028](https://github.com/symengine/symengine/pull/1028). Some minor changes still remain in some parts of the code. Most of the changes left are mainly related to code duplication, and will be fixed by writing their templated versions.

### Multivariate Polynomials

I started off by reading and understanding how the multivariate class is implemented currently in SymEngine. It was developed by the UC Davis team as a part of their course project. The basic idea is pretty simple. It still is a sparse representation and maps from a vector (representing the powers of each generator in the monomial) to the non-zero coefficient multiplied to it. We use an unordered map instead of the ordered map used in the univariate counterpart. The choice does make some sense, as deciding order between two monomials is subjective and does not make sense (for eg. which is larger `x**2*y` or `x*y**2`) Even if we define a custom ordering, there are no real benefits I could think of that it would provide.

On a side note, it kind of makes me think why we have stuck to the ordered map implementation for our univariate polynomials. The places it offers benefits are `eval` and `mul` (in some ways) I worked with ordered maps, as that was the implementation the initial polynomial class was built on. I'm also wondering if it is healthy to use two different implementations for both the polynomial types, univariate and multivariate. This needs to be discussed.

Right now, I'm basically refactoring most of the code written in the multivariate class, so that it matches the API more or less from the univariate case. Some functions have been re-written and some unnecessary storage within the polynomials have been removed. Initially, I thought I will stick with the same container based approach I used in the univariate case. This proves to be non trivial for the multivariate case. So, after some discussion with Isuru, we decided to stick with different implementations for SymEngine polynomials and Piranha polynomials, as combining code was not easy. The current work is in [#1049](https://github.com/symengine/symengine/pull/1049)

### Future Work

A lot is on my plate right now. Some of the work I plan to finish by the end of the next two weeks are

- Finish up the refactoring of multivariate polynomials, and make their interaction with univariate polynomials as seamless as possible

- Introduce wrappers for piranha polynomials to be used as multivariate polynomials within SymEngine

- Fix up the remainder of rational polynomials, and template code wherever possible to remove code duplicates

- Write conversions from `Basic` to multivariate polynomials, which will finish up one part of the coercion framework as proposed by Isuru

Off to work!