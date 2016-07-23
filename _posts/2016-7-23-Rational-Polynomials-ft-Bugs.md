---
layout: post
title: Rational Polynomials ft. Bugs
---

### Overview

Sorry I haven't been able to report my work for about two weeks now. Things have become slower mainly due to the fact that my university has resumed and along with it a heavily packed timetable and assignments in the first week don't help. I also caught a bad fever the past week which really hindered my progress, but it's dying down and I will resume my work with full vigor eventually.

The work I did do has been summarized below.

### Bug Fixes

While writing the code for the rational polynomials, as mentioned in the last blogpost, I encountered various bugs. Some of them caused other bugs to be exposed, which took a lot of time for me to debug.

- `pow(Poly, uint)` was throwing a segmentation fault. After digging in and wasting more than four hours on unrelated checks I figured out that the `eq` inside the polynomial class was incorrect. Without checking whether the other parameter was a `Poly` or not, I was `static_cast`ing it which posed a problem.

- The happiness was shortlived, as the bug persisted. On further inspection I found that the polynomial was being treated as a number! This was because `is_a_Number` relied on typecodes, and the polynomial types were defined before the `NUMBERWRAPPER` class, which deemed them numbers. The fix for this was simple, just move the polynomial type code definitions after the numbers.

- The `pow` tests pass, but what's this? All the `MSVC` builds on appveyor fail. They all fail a `coeff` test. Wow, I had not changed any code related to `coeff` at all, how does it affect that specific test and only on the `MSVC` compiler? This kept me wondering and looking at the source for a day. Finally, I had to login to the VM of appveyor running the tests. I was not familiar with windows development environment at all, which was the reason I failed a couple of times before I gave up debugging. The next morning I woke up determined to fix this Windows bug, I set the break points in Visual Studio and started the code execution. I found it! It was a bug in the `CoeffVisitor` itself. The code for the `coeff` function was incomplete. Why wasn't this bug being captured before? Probably because the previous change (in the typecodes) caused a reordering in  a map, which no other compiler was doing. Do read up [here](https://github.com/symengine/symengine/pull/1033#issuecomment-232973025) for more details.

- An appveyor build was failing for unknown reason, which had to be shifted to allowed failures

This was basically the components of [#1033](https://github.com/symengine/symengine/pull/1033). Less quantity of changes, but really important none the less.

### Rational Polynomials

The work with rational polynomials continues. I had underestimated the amount of work required, and I also feel that I should have broken down rational polynomials into three parts each, just like integer polynomials. Right now, the work continues in [#1028](https://github.com/symengine/symengine/pull/1028), but it's soon going to become huge with all varieties of changes.

### Miscellaneous

I finally benchmarked [cotire](https://github.com/sakra/cotire) to see how much speedup it was providing to SymEngine builds. [Here](https://github.com/symengine/symengine/issues/1023) is the short summary of the speedups obtained, and the work to include it is in [#1041](https://github.com/symengine/symengine/pull/1041).

Also a small bug was present in our flint and gmp rational number wrappers. We were not canonicalizing on construction from two integers. It was fixed in [#1031](https://github.com/symengine/symengine/pull/1031).

Laters!