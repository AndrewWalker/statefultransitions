+++
Categories = ["python", "scons"]
Description = ""
Tags = []
date = "2015-05-27T20:21:27+10:00"
title = "Stupid Things to do with SCons"

+++

*TL;DR* Don't use the Python callstack to find variables - it can lead to subtle problems for users of your code.

This post came about as part of an investigation into extending [SCons](http://www.scons.org/) functionality without having to alter the source code. The primary objective was to validate that the absolute path of an SConscript file exists, or provide (improved) feedback if it doesn't. This is important in situations where you're using variant build directories (debug and release) and your builds are spread over many directories.  This is one of the things SCons is particularly poor at reporting diagnostics for. 

If this was just Python, without the magic of SCons happening, this really would be trivial, in fact it's one of those things that Python excels at.

```python
import os
path = 'path/to/your/SConscript'
path = os.path.abspath(os.path.normpath(path))
if not os.path.exists(path):
    raise Exception('SConcript does not exist at %s' % path)
SConscript(path, ...)
```

Given enough SConscript files, this pattern is annoying and worthy of refactoring. One way to do that would be to pull that code out, and put it into an external module. Given that we'll always want to do this with our calls to SConscript, it's probably easier to adjust the SConscript method, rather than adding a second function. Altering the behavior of Python functions and methods at runtime by pre- or post- fixing code is most easily achieved by using [decorators](https://wiki.python.org/moin/PythonDecorators), which is really just a fancy name for a function that wraps another function.

A modern decorator would look something like this:

```python
# scommon.py
import os
import wrapt

@wrapt.decorator
def sconcript_validator(wrapped, instance, args, kwargs):
    path = args[0]
    path = os.path.abspath(os.path.normpath(path))
    if not os.path.exists(path):
        raise Exception('SConcript does not exist at %s' % path)
    return wrapped(*args, **kwargs)
```

Note that I've chosen to make use of the [wrapt](https://github.com/GrahamDumpleton/wrapt) which deals with all the ugly corner cases of decorators for me. 

This means that I can rewrite my SConstruct as:

```python
# SConstruct
import scommon 
menv = Environment()
menv.SConscript = scommon.sconcript_validator(menv.SConscript)
```

And at that point I should be able to run SCons as per normal, with a nicely updated SConscript function, and everything should be rainbows and unicorns.

Except it isn't. Instead SCons fails with the traceback

```shell
$ scons
scons: Reading SConscript files ...

scons: *** Export of non-existent variable ''menv''
File "/Users/username/tmp/scommon.py", line 10, in sconcript_validator
```

This occurs because SCons does much of it's heavy lifting by making use of the Python callstack, finding targets via introspection of the call frames. Unfortunately, the way it does this is somewhat fragile, expecting calling code to be a certain number of stack frames above (the internals of) SConscript. 

From SCons.Script.SConcript:

```python
# This is the magic line that actually reads up
# and executes the stuff in the SConscript file
# ...
call_stack[-1].globals.update({stack_bottom:1})
old_file = call_stack[-1].globals.get('__file__')
```

And eventually:

```python
exec _file_ in call_stack[-1].globals
```

The consequence of this is that, to the best of my knowledge, there is no easy way to decorate the SConscript method. 

All's not lost, the dynamic nature of Python means we can reach directly into the SConcript module, introspect data about the code, including getting the original source code (using [inspect](https://docs.python.org/2/library/inspect.html), rewrite the lines of that function, [exec](https://docs.python.org/2/reference/simple_stmts.html#exec) that string in the context of the original functions scope, before replacing the function back on the class with [setattr](https://docs.python.org/2/library/functions.html#setattr). The final result, while ugly is functional.

The [final result](https://gist.github.com/AndrewWalker/d9d74f5f46651c6607b4)
looks like the ugliest Python I've written for a long while. Somehow I don't
think this ones going to make it past code-review.


