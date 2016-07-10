---
layout: post
title: Rational Polynomials
---

### Overview

This last week was divided into two parts. The first half was spent in fixing miscellaneous issue with the last PR, regarding the basic to polynomial converisons. The second half was spent in thinking of how SymEngine should store rational polynomials (polynomials with rational coefficients) and how infact should they be implemented.

### Leftovers from Basic Conversions

As mentioned, the first half of my week was spent finalizing the basic to polynomial conversions PR. 

- Using `divnum` and `mulnum` methods instead of using `rcp_cast<Number>()` on `mul` and `div` respectively. These made the code much more readable too.

- Use `static_cast<const Rational &>(*rat_rcp).get_den()` instead of `rcp_static_cast<const Rational>(rat_rcp)->get_den()`. The logic behind this is, that in the second case, another `RCP` is being constructed, and the `get_den` function is called on this new RCP. This is unecessary and can be avoided using the former approach.

- At many places within the visitor pattern, I was calling the external function `basic_to_poly`. This was not wrong, but caused necessary overhead and recursion, which is not usually desired. Isuru suggested that I can reuse the visitor and successfully avoid the recursion. Here's an example

```
dict_type _basic_to_poly(basic, gen)
{	
	BasicToPolyV v;
	v.apply(basic, gen);
}

// dict is the answer we will be returning
void BasicToPolyV::bvisit(const Add &x)
{
    for (auto const &it : x.dict_)
    	// unnecessary overhead
        dict += _basic_to_upoly(it, gen);
}
```

becomes this

```
// dict is the answer we will be returning
void BasicToPolyV::bvisit(const Add &x)
{	
	dict_type res;
    for (auto const &it : x.dict_)
    	// no overhead
        res += BasicToPolyV::apply(it, gen);
    dict = std::move(res);
}
```

- There was also a [discussion](https://github.com/symengine/symengine/pull/1003#discussion_r69408888) on how the visitor for `Pow` in `find_gen_pow` was implemented. Isuru pointed out, how it would give wrong answers in the cases he put forward. But I had thought of other cases (like `2**(2**(x+1))`) which would would then not work, if the code was changed. These were special type of cases where the expression could be 'simplified'. There's some discussion on the issue [here](https://github.com/symengine/symengine/issues/1021).

### Starting Rational Polynomials




I hope to get all the rational polynomials ready and tested in the next two to three days.

Tata!