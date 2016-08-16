---
layout: post
title: Containers for Multivariate Polynomials
---

### Overview

After the last blog post, I started working on the "fixing up the remainder of rational polynomials". This also included adding support for basic to rational polynomial conversions. The current work is in [#1055](https://github.com/symengine/symengine/pull/1055) which has not been merged in yet.

Another thing I started working on was conversions from SymEngine symbolics to multivariate polynomials. The way the current conversions work, led me to change the overall structure of multivariate polynomials.

### Restructuring Multivariate Polynomials

The current working of basic conversions is that it constructs the "internal container" of the polynomial as it parses the expression. These containers have operators overloaded and they can easily be manipulated by the functions. Handling multivariate polynomials using this approach would be impossible with the current structure of the code. There were no "containers" per se in the previous implementation.

I thought it would be better if we can implement a container based polynomial structure for multivariate polynomials too. Fortunately, the way the SymEngine polynomials work, an approach similar to the univariate case of containers worked. The container is a wrapper around a map from vector to a value (the coefficient)

```c++
template <typename Vec, typename Value, typename Wrapper>
class UDictWrapper
{
public:
    using Dict = std::unordered_map<Vec, Value, vec_hash<Vec>>;
    Dict dict_;
    unsigned int vec_size;

    ...
```
The polynomial class uses this as a container, along with information about the generators and their ordering. Apart from holding the map, this class has various constructors, and all the operators overloaded for it. It also has helper functions, which help translate the current keys to another specific order/ arrangement. The container assumes that the all operations done to it are with other "adjusted" containers. Currently, the job of adjusting/aligning th container has been delegated to functions like `add_mpoly` etc.

```c++
template <typename Poly>
RCP<const Poly> add_mpoly(const Poly &a, const Poly &b)
{
    typename Poly::container_type x, y;
    set_basic s = get_translated_container(x, y, a, b);
    x += y;
    return Poly::from_container(s, std::move(x));
}
```

and 

```c++
template <typename Poly, typename Container>
set_basic get_translated_container(Container &x, Container &y, 
								   const Poly &a, const Poly &b)
{
    vec_uint v1, v2;
    set_basic s;

    unsigned int sz = reconcile(v1, v2, s, a.vars_, b.vars_);
    x = a.poly_.translate(v1, sz);
    y = b.poly_.translate(v2, sz);

    return s;
}
```

Basically, both the containers get the ordering of generators in the "new" polynomial which will soon be constructed, and adjust the ordering of the vectors inside their maps accordingly. Thus consistency is maintained. All the work done is in [#1049](https://github.com/symengine/symengine/pull/1049) and this has been merged in.

### Remaining work

 - After getting this restructure done, I started work on the basic to multivariate conversions, though I have not been able to cover it completely. The idea for these conversions remains the same and should be easy to implement.

 - I also could not spend time to integrate wrappers to Piranha polynomials which can be used as multivariate polynomials within SymEngine

### Wrapping up GSoC

This is officially the last week of Google Summer of Code '16. Although GSoC is over, I plan to get the above mentioned work finished within August, before any other big undertaking. It has been a wonderful journey of insight and knowledge. Thanks a lot to my mentors Isuru, Ondrej and Sumith for helping me throughout the summers, and always being there for discussions and the silly issues I bombarded them with!

Bye!