+++
Categories = []
Description = ""
Tags = ["python", "matlab"]
date = "2015-05-19T21:35:28+10:00"
title = "Making numpy's spacing more like MATLAB's eps"

+++

Anyone who has worked with scientists knows the tension between MATLAB and
Python, the oft-repeated closed vs open source debate. For those who straddle
the gap, it's important to know where to go to get help when you're moving
between the two worlds.  From a Python perspective, the most upto date
reference is [Numpy for MATLAB
users](http://wiki.scipy.org/NumPy_for_Matlab_Users) on the scipy wiki.

During the process of porting the [conformal mapping
toolkit](https://github.com/AndrewWalker/cmtoolkit), one of the issues that
came up related to MATLAB's [eps](mathworks.com/help/matlab/ref/eps.html)
function. This function determines what the interval to the next floating point
number is - an important prerequisite in doing approximately equal tests on
floating point numbers.

The guide (correctly) describes that `eps` is equivalent to the numpy
[ufunc](http://docs.scipy.org/doc/numpy/reference/ufuncs.html) `np.spacing(1)`.
More information about exactly what spacing does is provided by the numpy
`info` function:

```python
import numpy as np

np.info(np.spacing) 
```

The documentation states that the ufunc returns "the distance between x and the
nearest adjacent number".  However, MATLAB`s eps seems to differ from spacing
in two important ways.  Firstly, the values it returns are always positive
(it's the *absolute*) spacing information. Second, and more importantly, it
also deals with complex values, which is critical for conformal mapping.

So a slightly revised version of `eps` for python might read:

```python
def eps(z):
    """Equivalent to MATLAB eps
    """
    zre = np.real(z)
    zim = np.imag(z)
    return np.spacing(np.max([zre, zim]))
```
