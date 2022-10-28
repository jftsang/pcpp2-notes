# Some of my favourite decorators

Comments and corrections to [J. M. F. Tsang](j.m.f.tsang@cantab.net).

---

In this piece I describe how function decorators work, and give some examples of
little useful decorators that I like to use all the time.


## Decorating functions: an overview

When functions (and classes) are declared they can be _decorated_. The syntax is
to place a `@` followed by the name of the decorator immediately before the
`def` line:
```python
@func_dec
def func(arg):
    do_stuff(arg)
```
Syntactically, this is equivalent to:
```python
def func(arg):
    do_stuff(arg)

func = func_dec(func)
```
And it is similar to:
```python
def undecorated_func(arg):
    do_stuff(arg)

func = func_dec(undecorated_func)
```
except that the name `undecorated_func` doesn't get created or modified.

Here, `func_dec` should be a _higher-order function_ (hof): that is, a function
that takes a function and returns another function, possibly with side effects.
The syntax for creating such hofs is a little messy (examples below), but the
basic idea is that when we define `func`, we first define its basic behaviour,
but then immediately 'do something else with it or to it'.

There are two important reasons for decorating functions. The first is to modify
or augment a function to give it additional desirable behaviour, such as a side
effect. The second is to register the fact that a function has been defined,
perhaps in a list or dictionary that will be used later.


## Example 1: Timing

For example, a decorator that makes a function report how much time it took to run:
```python
from functools import wraps
from time import time

def timed(f):
    @wraps(f)
    def decorated_f(*args, **kwargs):
        tic = time()
        try:
            return f(*args, **kwargs)
        finally:
            toc = time()
            print(f"Elapsed time was {toc - tic} seconds")

    return decorated_f
```
(And yes, it would be more proper to use a logger rather than `print`.)

So, `timed` is a function that takes a function `f` as its parameter, and
returns a function `decorated_f`. The function `decorated_f` passes its
arguments to `f` and returns the return value of `f`, but it does stuff in
addition to `f` itself.  Before calling `f`, it takes the time and stores it in
a (local) variable `tic`.  After running `f`, but before returning, in its
`finally` clause it takes the time again, stores this in a variable `toc`, and
then reports the elapsed time `toc - tic`.

The `wraps(f)` before the (itself a decorator!) is needed to make sure that the
decorated function has the correct name and docstring: see [this Stack Overflow
thread](https://stackoverflow.com/questions/308999/what-does-functools-wraps-do#309000)
for more details.

Example usage:
```python
@timed
def my_function(x):
    """For now, just pretend that this is a
    tough calculation that might take a long
    time.
    """
    return x * x * x
```

The real benefit comes when we have several functions `f`, `g`, `h`, _etc._ that
all need to be timed. By placing the code for timing and reporting the time into
the decorator, all we need to do is to precede each of their definitions by
`@timed`. We don't need to repeat this boilerplate inside each of the
definitions of `f`, `g` and `h`, and so they can focus on the actual mechanics.

For example, to compare the speeds of two different sorting algorithms:
```python
@timed
def bubblesort(lst):
    ...

@timed
def quicksort(lst):
    ...

lst = [5, 4, 6, 1, 21]
bubblesort(lst)
quicksort(lst)
```


## Example 2: Tee

When debugging a function, it's often useful to see its return value, especially
if this value is then being used somewhere else. The following decorator causes
a function to print out its return value in addition to returning it.
```python
def tee(f):
    def decorated_f(*args, **kwargs):
        r = f(*args, **kwargs)
        print(r)
        return r

    return decorated_f
```
When debugging `f`, you can simply put `@tee` above the definition of `f`.


## Example 3: Overloading and polymorphism and `@np.vectorize`

One thing I dislike about Python is that, unlike statically typed languages such
as C++ and Java, it is not possible, out-of-the-box, to create multiple versions
of a function with the same name but different behaviours depending on the type
of the input.


## Parameterising decorators

In the examples so far the decorator has just been a single function that
doesn't take any parameters. However, it is usually useful to be able to
parameterise the decorator. For example, a more sophisticated version of the
`@tee` decorator might supporting writing not only to `stdout` (using `print`)
but also to another file. This would be useful for functions that produce a
large amount of output.
```python
@tee("output.txt")
def f(x):
    return x * x
```
The idea now is that the decorator `tee("output.txt")` should still be a
function that takes a function and returns a function.  This can be achieved in
two ways. Either `tee` can be a function that takes in a string and returns a
function, so that `tee("output.txt")` is a function:
```python
def tee(filename):
    def decorator(f):
        @wraps(f)
        def decorated_f(*args, **kwargs):
            try:
                r = f(*args, **kwargs)
                return r
            finally:
                with open(filename, "w") as fp:
                    fp.write(r)

        return decorated_f
    return decorator
```

Alternatively, and more neatly, we could use a *class-based decorator* (and we
use TitleCase for class names). Consider first:
```python
class Tee:
    def __init__(self, filename):
        this.filename = filename
```
so that `Tee("output.txt")` initialises an object (same syntax as a function
call). This object must be callable (remember, decorators are functions of
functions), so we must also equip `Tee` with a `__call__` method:
```python
class Tee:
    ... # as above

    def __call__(f):
        @wraps(f)
        def decorated_f(*args, **kwargs):
            ...  # as above
        return decorated_f
```


## Example 4: Registering functions as they are defined
Remember that decorating a function
```python
@dec
def f(x):
    ...
```
is equivalent to first defining the function `f` and then immediately applying a
decorator `dec`, _i.e._
```python
def f(x):
    ...

f = dec(f)
```
The decorator `dec` is allowed to have side effects when it is called. Invoking
these side effects may be interesting, even if `dec` returns `f` without
modifying its behaviour.

Here's a really boring application:
```python
def announce(f):
    print(f"You have just defined {f.__name__}")
    return f

@announce
def spam(x):
    ...

@announce
def eggs(x):
    ...
```
In this example, the messages get printed as the functions `spam` and `eggs` are
_defined_, not as they are called.

A more useful application would put the decorated function `f` into some sort of
dictionary or other collection. Registering a function when it is defined is
actually how the Flask framework (and many others) register URLs: see [this blog
post by Ains](https://ains.co/blog/things-which-arent-magic-flask-part-1.html).


## Applying multiple decorators

Just stack them. This lets us combine the effects (hopefully benefits) of
different decorators.

```python
@timed
@tee
def quicksort(lst):
    ...
```
The decorators are applied in order, with the bottom one first. So, the above is equivalent to:
```python
quicksort = timed(tee(quicksort))
```
Order matters when the decorations are used to register a function. For example, suppose a decorator `@register` is used to register functions into a set (*e.g.* in Flask, to add the function as a route for the app). The following have different behaviour:
```python
REGISTRY = set()

def register(f):
    @wraps(f)
    def decorated_f(*args, **kwargs):
        REGISTRY.add(f)
        return f(*args, **kwargs)

@register
@tee
def f():
    ...

@tee
@register
def g():
    ...
```
In the first example, the registered function includes the `@tee` behaviour; in
the second, it does not.


## Example 5: Suppressing exceptions

Suppose you have a function `func` consisting of a sequence of operations that
might raise a `ValueError` (say), in which case you want the function to have
return a default value, such as `None`. You might wrap a try-except block around
the body of your function, like this:
```python
def func(x):
    try:
        return risky_operation(x)
        # a whole load of stuff that
        # could raise a ValueError
    except ValueError:
        return None
```
All the `except ValueError` does is to catch the possibility of a `ValueError`
and just return `None`.

Instead of having the `try` around the whole body of the function, with an extra
layer of indent around everything, it is possible to do the exception handling
in a decorator instead. Again, this is especially useful if this is a pattern
that you will use in more than one function.
```python
def quiet(f):
    @wraps(f)
    def decorated_f(*args, **kwargs):
        try:
            return f(*args, **kwargs)
        except ValueError:
            return None

    return decorated_f


# Then just define your function like this
@quiet
def func(x):
    return risky_operation(x)
```
This is probably too specific to be useful. Perhaps you might be interested in
other types of errors; perhaps you might want to have some other default return
value. We can generalise this by parameterising the decorator.


## Decorating classes

Classes can also be decorated, for the same reasons you may want to decorate
functions.
```python
@cls_dec
class MyClass:
    pass
```
is equivalent to:
```python
class MyClass:
    pass

MyClass = cls_dec(MyClass)
```
The function `cls_dec` is another hof, but this time it takes a class as its
argument, and returns a class, possibly with side effects.

Decoration could be used to extend a class, although inheritance is probably
better suited for this. Decorating classes is most useful for registering the
class.
