# Design Patterns

Comments and corrections to [J. M. F. Tsang](j.m.f.tsang@cantab.net).

---

## Introduction

Design patterns are general, abstract, high-level strategies for structuring an
application.  They are common, 'reusable' structures that are applicable to
problems at different levels, usually a component of a larger application; yet
largely independent of the details of the application. The patterns described
here are most applicable to object-oriented languages and frameworks, and may
not be suitable for problems that would be better tacked with a functional
approach.

Wikipedia says ([permalink](https://en.wikipedia.org/w/index.php?title=Software_design_pattern&oldid=1028423053)):

> In software engineering, a *software design pattern* is a general, reusable
> solution to a commonly occurring problem within a given context in software
> design. It is not a finished design that can be transformed directly into
> source or machine code. Rather, it is a description or template for how to
> solve a problem that can be used in many different situations. Design patterns
> are formalized best practices that the programmer can use to solve common
> problems when designing an application or system.
>
> Object-oriented design patterns typically show relationships and interactions
> between classes or objects, without specifying the final application classes
> or objects that are involved. Patterns that imply mutable state may be
> unsuited for functional programming languages. Some patterns can be rendered
> unnecessary in languages that have built-in support for solving the problem
> they are trying to solve, and object-oriented patterns are not necessarily
> suitable for non-object-oriented languages.

## Singleton pattern

A **singleton** is a class of which there may be at most one instance throughout
the application; and all parts of the application must have access to that one
instance.

### Why

Configurations or resources (_e.g._ database or network connections, disk caches) are often held in global variables or functions, to ensure consistency across usages as well as minimizing expensive operations, _e.g._ by making only a single database connection and reusing it in different areas. To avoid cluttering the module namespace, we can encapsulate them into a class that contains these variables and functions, and we make this class a singleton so that these resources are preserved rather than creating a new instance each time it is needed.

### How
The key is to have an instance of the class itself as one of its members, and to
An example of a singleton class in Python:
```python
class SingletonExample:
    __instance = None

    def __new__(cls):
        if cls.__instance is None:
            cls.__instance = object.__new__(cls)

        return cls.__instance

    # then other methods
```
This class, `Singleton`, has a class member `Singleton.__instance` that is
initialised to `None`. Then when there is an attempt to create an instance of
`Singleton`, the constructor `__new__` actually creates an instance if and only
if such an instance does not already exist. Otherwise, it just returns the
existing instance. Thus:
```python
s1 = SingletonExample()
s2 = SingletonExample()
assert s1 is s2  # not just s1 == s2
```
Since `s1` and `s2` are actually the same instance, mutating `s1` will similarly mutate `s2`. Note that singleton-ness does _not_ imply that the class is immutable!
```python
s1.foo = 'boo'
print(s2.foo)  # doesn't AttributeError
```

Some technical points:
  1. The use of `__new__` instead of `__init__` is explained at [this Stack Overflow answer](https://stackoverflow.com/a/4859181).
  2. When subclassing, instead of `object.__new__` it may be more
     appropriate to call `super().__new__` (which in turn may call
     `object.__new__`).  However, [you probably shouldn't be subclassing a singleton class](https://softwareengineering.stackexchange.com/questions/372196/is-it-worthy-subclassing-a-singleton-class).
  3. The use of double leading underscores in the name `__instance`
     triggers [name mangling](https://stackoverflow.com/a/34903236),
     essentially so that subclasses of `SingletonExample` will have have a
     different instance. But again, consider whether subclassing is
     really necessary.

### Application
```python
import warnings
import psycopg2

class Connection:
    __instance = None

    def __new__(cls, *args, **kwargs):
        if cls.__instance is None:
            print(f'Initializing a new connection with {args} and {kwargs}')
            cls.__instance = psycopg2.connect(*args, **kwargs)
            # For demonstration purposes you can just use a dict
            # cls.__instance = kwargs

        else:
            if args or kwargs:
                warnings.warn(
                    'A connection has already been initialized, '
                    'so new arguments are ignored.'
                )
            print('Reusing an already-initialized connection instance')


        return cls.__instance
```

### Alternatives
When dealing with only one or two expensive functions and using names at the module namespace is not a problem (_e.g._ if the connection class is in a separate module), then it is simpler to decorate those functions with `@functools.lru_cache`.
```python
from functools import lru_cache
import psycopg2

@lru_cache()
def connect(*args, **kwargs):
    return psycopg2.connect(*args, **kwargs)
```


## Delegate pattern



## Factory pattern

In _Design Patterns: Elements of Reusable Object-Oriented Software_, Gamma _et
al._ wrote of the factory pattern:

> Define an interface for creating an object, but let subclasses decide which
> class to instantiate. The Factory method lets a class defer instantiation it
> uses to subclasses.

### How

Suppose that `Spam` and `Lobster` are two subclasses of a base class `Food`.
Consider this example:
```python
from abc import ABC, abstractmethod

class FoodFactory(ABC):
    """A factory for food, i.e. a restaurant"""
    @abstractmethod
    def cook(self) -> Food:
        """Define an interface for creating an object [of type Food]..."""
        raise NotImplementedError

class Diner(FoodFactory):
    def cook(self) -> Spam:
        """...but let subclasses [of the interface] decide which class
        [i.e. which subclass of Food] to instantiate [and return].
        """
        return Spam()

class FancyRestaurant(FoodFactory):
    def cook(self) -> Lobster:
        """A different subclass of the interface might instantiate a
        different class of object.
        """
        return Lobster()
```
According to the documentation of
[`abc.abstractmethod`](https://docs.python.org/3/library/abc.html#abc.abstractmethod),
marking a method as an abstract method, using the decorator `@abstractmethod`,
has this effect:
> A class that has a metaclass derived from `ABCMeta` [therefore including
> `ABC`] cannot be instantiated unless all of its abstract methods and
> properties are overridden.

If `Diner` and `FancyRetaurant` had not implemented `cook`, then attempting to
initialise one with `Diner()` would result in a `TypeError`.

### Why

The basic idea is that `Diner.cook` and `FancyRestaurant.cook` are two different
methods, each of which is used to construct a new instance of `Food`. This means
it is possible to create a function that can take any sort of `Food` and 

The factory pattern is most useful for object-oriented applications: the
abstraction allows one to create functions that can use any sort of
`Restaurant`. This is particularly useful in staticly-typed languages. (We
include type hints in these examples, but Python does not enforce them.)
```python
def consume(food: Food) -> None:
    pass
    

def dine_out(restaurant: FoodFactory) -> None:
    food = restaurant.cook()
    consume(food)
```
Here, the function `dine_out` can take either sort of `FoodFactory`, or indeed
any object that has a `cook` method.

### Factories as functions

The factory pattern is perhaps less useful in Python when a more functional
approach is desired. We could dispense with the classes `FoodFactory`, `Diner`
and `FancyRestaurant` and instead use methods directly. Consider the following
alternative:
```python
def dine(cook: Callable[[], Food]) -> None:
    """You don't have to eat at a restaurant."""
    consume(cook())
```
This takes as its input a callable (probably a function), `cook`, and calls it.
The function `dine` can take any input for `cook`, so long as it is a function
that takes no arguments and produces a `Food`. For example:
```python
dine(cook_spaghetti)
dine(cook_salmon)
```
Note that we pass in the functions themselves as arguments; we don't do
`dine(cook_spaghetti())`.

## Facade pattern

Wikipedia says of the facade pattern ([permalink](https://en.wikipedia.org/w/index.php?title=Facade_pattern&oldid=996427370)):

> Analogous to a facade in architecture, a **facade** is an object that serves as a
> front-facing interface masking more complex underlying or structural code. A
> facade can improve the readability and usability of a software library by
> **masking interaction with more complex components behind a single (and often
> simplified) API**. [...]
> 
> Developers often use the facade design pattern when a system is very complex
> or difficult to understand because the system has many interdependent classes
> or because its source code is unavailable. This pattern hides the complexities
> of the larger system and provides a simpler interface to the client. It
> typically involves a single wrapper class that contains a set of members
> required by the client. These members access the system on behalf of the
> facade client and hide the implementation details.

An API is a common example of a facade: the client does not need to know (and
perhaps should not be permitted to know) the machinery of the server as it
processes a request.

### Encapsulation

The facade pattern is closely linked to the notions of encapsulation: the
properties of an encapsulated object may be queried or set, but the underlying
implementation of those properties should not be exposed.

Consider for example:
```python
class Point2D():
    def __init__(self, x, y):
        self.x = x
        self.y = y
        
    def distance_from_origin(self):
        """Return the distance of the point from the origin."""
        return (self.x ** 2 + self.y ** 2) ** 0.5
``` 
In this class, the position of the point has been stored in terms of two
separate variables for the Cartesian coordinates of the point. If we are
interested in the distance of the point from the origin, we can call
```python
p = Point2D(3, 4)
print(p.distance_from_origin())
```
The distance of a point from the origin is a geometric property that does not
depend on how the point `p` is represented (it could alternatively be stored as
a complex number, or as a 2-tuple, or in terms of polar coordinates, maybe even
queried from a database somewhere) By defining a method `distance_from_origin`,
we can change how we internally represent `p` and then change the definition of
`distance_of_origin`, without needing to change any code that uses this method.

If instead the other code calculated the same quantity directly:
```python
print((p.x ** 2 + p.y ** 2) ** 0.5)
```
this code would need to be changed if the way `p` is stored is changed.


## Proxy pattern

A **proxy** is a class that provides an interface to something else (such as
another class, or something lower-level such as a network connection or a
device), possibly introducing additional functionality, or controlling access.
In this way it plays a similar role to a facade: the proxy introduces an
additional layer of abstraction.

### How

Given a class `C`, a proxy class `ProxyC` for `C` could be defined as a subclass
of `C` that overrides one of its methods. For example, suppose `C` has a method
`func` that we want to control access to, requiring a password.

```python
from getpass import getpass

class C:
    def __init__(self, pow):
        self.pow = pow
        
    def func(self, x):
        """We want to restrict access to this function."""
        return x ** pow
        
class ProxyC(C):
    def __init__(self, password, *args, **kwargs):
        self.password = password
        super().__init__(*args, **kwargs)
        
    def func(self, x):
        if getpass() != self.password:
            raise PermissionError
        return super().func(x)
```
To introduce password protection, we need only change existing code like this:
```python
c = C(3)
c.func(6)
```
to this:
```python
c = ProxyC("secret phrase", 3)
c.func(6)
```

Alternatively, a proxy class could be defined by creating a separate class that
has an instance of `C` as one of its member variables.
```python
class ProxyC:
    def __init__(self, password):
        self.c = C()
```




## Observer pattern


## Command pattern


## Template Method pattern


## Model-View-Controller (MVC)


## State Design pattern
