---
layout: post
title: "Numpy changes in 1.10"
description: ""
category: 
header-img: "img/2.jpg"
tags: ["python", "windows"]
---

<h2 class="section-heading">Numpy changes in 1.10</h2>

Casting behavior is different in numpy v1.10, so if you've gotten away with 'unsafe' casting up until now, you are in for a rude awakening with numpy v1.10. 

Previously, this kind of code was OK:

{% highlight python %}
>>> import numpy as np 
>>> x = np.array([1000], dtype='int64')
>>> x *= 1.02
>>> x
array([1020]) 
{% endhighlight %}

This is because the default casting behavior was "unsafe", so it was OK by default to take the floating point result of the multiplication and store it back into an integer dtype. In Numpy 1.10, the behavior has changed to a default rule for casting called "same_kind". This allows one to, by default, cast a float64 to, say, a float32, with no error generated, but not from float64 to int64. Here's an excerpt from the [release notes from 1.9](http://docs.scipy.org/doc/numpy/release.html) detailing what's going to change in 1.10:

>In a future version of numpy, the default casting rule for UFunc out= parameters will be changed from ‘unsafe’ to ‘same_kind’. (This also applies to in-place operations like a += b, which is equivalent to np.add(a, b, out=a).) Most usages which violate the ‘same_kind’ rule are likely bugs, so this change may expose previously undetected errors in projects that depend on NumPy. In this version of numpy, such usages will continue to succeed, but will raise a DeprecationWarning.

My usage of this feature was intentional, but I confess it was probably not the safest behavior to rely on.

To get the same behavior in v1.10 that you got with numpy v1.9 and earlier, you have to now explicitly specify that you want the unsafe casting behavior. This seems reasonable, but there's no way to spell that out with the inplace syntax, so you have to use an actual function call with the 'out' parameter. With that syntax, you can now stick in the desired keyword argument:

{% highlight python %}
>>> import numpy as np 
>>> x = np.array([1000], dtype='int64')
>>> np.multiply(x, 1.02, out=x, casting='unsafe') 
>>> x
array([1020])
{% endhighlight %}

This is backwards compatible syntax that actually does make it more explicit that you are chopping off a floating point value to put it in an int type. It's a bit uglier, in my view, but that's pretty much the only downside I can see.

Ok, numpy, you win. I humbly accept your reproof and promise to be more explicit about my unsafe casts in the future (since I basically have no choice!).
