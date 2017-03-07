---
title: 'A Faster "atan()" Function'
author: marco.lizza
layout: post
permalink: /a-faster-atan-function/
thumbnail: flask
comments: true
categories:
  - snippets
tags:
  - optimization
---
Optimizing a function/algorithm/procedure is never an easy task. Luckily there's a whole bunch of cases where the *almighty LUT* (**L**ook-**U**p **T**able) makes it possible (and frequently in an easy way). The `atan()`  function (which alongside `atan2()` is quite a neat and useful, albeit underrated, function indeed) can be optimized by means of a LUT.

In it's simplest incarnation the Look-Up Table is an array that serves as a *memoization cache* for (pre)calculated results of a function of chose. Let's see an example with the `sin()` trigonometric function.

```c++
double _sin[360];
for (int i = 0; i < 360; ++i) {
    double radians = static_cast<double>(i) / 57.295779513082320876798154814105;
    _sin[i] = sin(radians);
}
```

So, considering single one degree steps, we have pre-calculated the whole (periodic) function values. Anytime we need compute `y = sin(x)` we simply access the array as `_sin[x % 360]`, with `x`  expressing the angle in degrees. We would call this method *forward access*, since we use the independent variable (or a function of it) as an index for the LUT to obtain the dependent variable (i.e. the result).

Arrays in C/C++ are handled efficiently thanks to the pointer arithmetic mechanism, and trigonometric functions can easily optimized with LUTs. Sounds easy, uh? Well, the single most difficult thing is picking the correct size for the LUT itself. We could also have expressed the angle in radians, but is would complicate the LUT indexing access: indexes need to be integer values while radians aren't, so a scaling factor need to be introduced (e.g. access the LUT by 1/100th radian steps). To be honest, a scaling factor would be needed also in the case we wanted a finer resolution (e.g. 1/10th degrees unit).

The `atan()`  function maps values in the whole real numbers domain to values in the `-PI/2` and `+PI/2` range (boundaries excluded). Since the independent variable range is infinite we need to proceed the other way out and use the LUT in what we call a *reverse access* method. First of all we define the LUT size basing of the dependent variable range: a valid choice is the following:

```c++
// The range spans 180 degrees, but we want it to be centered and symmetric
// with reference to the "zero" angle.
int _atan[179];

for (int i = 0; < 179; ++i) {
    double degrees = static_cast<double>(i - 89);
    double radians = degrees / 57.295779513082320876798154814105;
    _atan[i] = static_cast<int>(tan(radians) * 1024.0);
}
```

> Note, that we are also getting rid of floating-point variables as their slowness is (in)famous among mobile-device developers, in favour of `22:10` fixed-point integers.

Once the LUT is ready, to calculate `y = atan(x)`  we find the index `i` such as `_atan[i] < y < _atan[i + 1]`. To accomplish this we need to scan the LUT content; since we filled with ever-increasing values this will work perfectly. A simple access it as described for the `sin()` function won't work.

```c++
// [yx] need to be in fixed-point 22:10 format, so you may need to pre-multiply
// the value prior calling the function.
int atan(int yx)
{
    if (yx < _atan[0]) {
        return -90;
    } else
    if (yx >= _atan[178]) {
        return 90;
    } else {
        for (int i = 1; i < 178; ++i) {
            if (yx < _lut[i]) {
                return i - 90;
            }
        }
        return -88; // -89 will never be returned
    }
}
```

We can improve the algorithm by implementing a binary-search scan when accessing the LUT. In this case we define an empirical surrogated *recursion base limit* (even though the algorithm is iterative in nature) to stop the divide-and-branch part and go for a brute-force LUT scan (but since now is performed on a smaller part of the array its cost should be negligible)

```c++
#define BASE_LIMIT 25

int bsearch(int value, int *vector, int length)
{
    int start = 0, end = length - 1;
    while (start < end) {
        int span = end - start + 1;
        if (span < BASE_LIMIT) {
            break;
        }
        int pivot = (start + end) / 2;
        if (value < vector[pivot]) {
            end = pivot;
        } else
        if (value > vector[pivot]) {
            start = pivot;
        } else {
            return pivot;
        }
    }
    for (int i = start + 1; i < end; ++i) {
        if (value < vector[i]) {
            return i - 1;
        }
    }
    return end - 1;
}

int atan(int yx) {
    if (yx < _atan[0]) {
        return -90;
    } else
    if (yx > _lut[178]) {
        return 90;
    } else {
        return bsearch(yx, _atan, 179) - 89;
    }
}
```

If degrees unit is not enough for the application's need, we could go for 1/10th of 1/100th of degrees. The LUT size will increase and with such a growth in size the binary-search optimization will be crucial; otherwise, the LUT-based algorithm will end up less efficient (or, let's say, comparable efficient) than the standard, unoptimized, algorithm.