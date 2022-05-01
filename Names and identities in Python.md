# Names and identities in Python

Comments and corrections to [J. M. F. Tsang](j.m.f.tsang@cantab.net).

---

## Everything is an object

In Python, **everything is an object**: be it an instance of a primitive type such as `int`, `float`, `list` or `str`; or an instance of a class; or the class itself; or a function; or even a module.
```python
class Class:
    def method(self):
        return 0


def fun():
    return 0


# All of these are True
print(isinstance(1, object))
print(isinstance('my string', object))

x = 1
print(isinstance(x, object))

print(isinstance(range, object))  # builtin
print(isinstance(fun, object))  # custom function
print(isinstance(int, object))  # builtin type
print(isinstance(Class, object))  # custom class
print(isinstance(Class.method, object))

instance = Class()
print(isinstance(instance, object))
print(isinstance(instance.method, object))

# Even modules are objects
import unittest
print(type(unittest))  # module
print(isinstance(unittest, object))  # True
```
All types, including builtin types and custom classes, are subtypes of this most basic type `object`.
```python
print(int.__bases__)
print(Class.__bases__)
```


## Names and assignment

Certain objects of primitive types such as `int` can be referenced directly as literals (`0`, `-5`), but most often we work with objects by giving them names ('variables'). Since 'everything is an object', all names are treated equally: any name can be used to refer to any type of object.

An assignment statement ([docs](https://docs.python.org/3.10/reference/simple_stmts.html#assignment-statements)) such as
```python
x = sin(3.14)
```
where the left hand side is a name, means 'evaluate the right hand side, then let the name on the left hand side now refer to that object'. So, the RHS is evaluated to produce some object of type `float`, and then the name `x` is bound to it. 

(Similarly for multiple assignments, like 
```python
x, y, z = 1, 2, 3
```
Assignments where the LHS is not a name, like
```python
obj.property = 1.23
dic['key'] = 'foo'
```
work a little differently...)

**Names are references to objects. Names are not the objects themselves.**

An object can have more than one name, and you can use `x is y` to test whether the names `x` and `y` point to the same object, *i.e.* are identical. 
```python
# Create two distinct instances of object
a = object()
b = object()
print(a is b)  # False
print(id(a), id(b))  # different id

# On the other hand:
c = object()
d = c
print(c is d)  # True
print(id(c), id(d))  # same id
```
Since `c` and `d` are the same object, operations that modify `c` are reflected in `d` as well. Consider for example:
```python
class Class:
    def __init__(self, x):
        self.x = x


c = Class(2)
d = c
print(c.x, d.x)  # 2 and 2
c.x = 99
print(c.x, d.x)  # 99, 99
```


## Identity

Each object has a unique identity, calling the builtin function `id` ([doc](https://docs.python.org/3/library/functions.html?#id)):
```python
print(id(100))
```
The test `x is y` is equivalent to testing whether `id(x) == id(y)`.

The `id` of an object has no intrinsic meaning, other than to uniquely and invariantly identify that object. (In CPython, the most common Python implementation, `id` gives the memory address of that object.) 

The identity of an object does not change over the lifetime of the object, even if the object is mutable and its keys or attributes change. In the following, each `print` statement will show that `id(d)` remains the same despite the changes being made to `d`. 
```python
d = dict()
print(id(d), d)

d['x'] = 1  # add a key
print(id(d), d)

d['x'] = 2  # overwrite a key
print(id(d), d)

d.update({'y': 1})  # call a method that mutates the dictionary
print(id(d), d)

d.pop('x')  # delete a key
print(id(d), d)
```
However, a reassignment of the name `d` will change `id(d)`.
```python
d = {'x': 1, 'y': 2}
print(id(d), d)

d['x'] = 3
print(id(d), d)  # id(d) unchanged

# However!
d = {'x': 3, 'y': 2}
print(id(d), d)  # id(d) changed
```
This is because `d` now refers to a new dictionary - although its contents are the same as the previous dictionary's.


## Equality and identity

The operation `x == y` tests for *equality*. Two different objects may be *equal* without being *identical*.
```python
x = 'Hello world'
y = 'Hello world'
print(x == y)  # True
print(x is y)  # False
print(id(x), id(y))  # different
```
In this example, in each of the two definitions a new `str` is being created, and the names `x` and `y` are being assigned to two different objects. 

For certain immutable literals Python will reuse the same underlying object. This includes the empty string `''`, the empty list `[]`, `None`, and small integers (depending on the implementation). 
```python
x = 16
y = 16
print(x is y)  # True

x = ''
y = ''
print(x is y)  # True
```


## Arguments

In Python, **function arguments are passed by name**. When an object is used as an argument to a function call, a reference to that object is passed into the function.
```python
def identity(x):
    print(id(x))
    return x
    

arg = object()
print(id(arg))

ret = identity(arg)  # prints the same id 
print(id(ret))  # prints the same id

print(arg is ret)  # True
```
This is different from some other languages that use 'pass by value', where a copy of the object is made before passing in.

Passing arguments by name (reference) means that mutations to an argument affect that object outside the scope of the function as well.
```python
def set_foo_key(dic):
    """Add or update the 'foo' key."""
    dic['foo'] = 'New value'


d = {'a': 1, 'b': 2}
set_foo_key(d)
print(d)  # {'a': 1, 'b': 2, 'foo': 'New value'}
```
One way to avoid this is by making a copy of the input before use ([doc](https://docs.python.org/3/library/copy.html) for `copy`).
```python
from copy import copy


def modify_copy(dic):
    dic2 = copy(dic)
    dic2['foo'] = 'New value'
    print(dic2)


d = {'a': 1, 'b': 2}
modify_copy(d)  # {'a': 1, 'b': 2, 'foo': 'New value'}
print(d)  # {'a': 1, 'b': 2}

```
