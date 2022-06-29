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

Configurations or resources (_e.g._ database or network connections,
disk caches) are often held in global variables or functions, to ensure
consistency across usages as well as minimizing expensive operations,
_e.g._ by making only a single database connection and reusing it in
different areas. To avoid cluttering the module namespace, we can
encapsulate them into a class that contains these variables and
functions, and we make this class a singleton so that these resources
are preserved rather than creating a new instance each time it is
needed.

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
Since `s1` and `s2` are actually the same instance, mutating `s1` will
similarly mutate `s2`. Note that singleton-ness does _not_ imply that
the class is immutable!
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

When dealing with only one or two expensive functions and using names at
the module namespace is not a problem (_e.g._ if the connection class is
in a separate module), then it is simpler to decorate those functions
with `@functools.lru_cache`.
```python
from functools import lru_cache
import psycopg2

@lru_cache()
def connect(*args, **kwargs):
    return psycopg2.connect(*args, **kwargs)
```


## Delegate pattern

In the **delegate** or **delegation pattern**, an object $A$ receives a
request and _delegates_ the work to some other object $B$, passing
alongside the _context_ of the request. In other words, $B$ has
'knowledge' of (at least part of) the state of $A$. This could be done
perhaps by $A$ including a reference to itself in its request to $B$, or
perhaps by $B$ having a reference to $A$ as an instance variable.

Consider the following implementation of a particle simulation, in which
particles exert forces on each other. The `Simulation` object maintains
a list of `Particle` objects, each of which has an `.update()` method.
When `simulation.update(dt)` is called, the `.update()` method is called
on each of the `Particle` objects. To do the update, each `Particle`
needs to know the forces on it from all the other particles, so it
maintains a reference to the `Simulation` to which it belongs.

(This is a terrible integration scheme! Don't do it!)
```python
class Simulation:
    def __init__(self, particles):
        for particle in particles:
            particle.simulation = self

        self.particles = particles
        self.time = 0

    def update(self, dt):
        self.time += dt
        for particle in self.particles:
            particle.update(dt)


class Particle:
    def __init__(self):
        ...  # mass, position, velocity
        self.simulation = None

    def calculate_force(self):
        force = (0, 0, 0)
        for other in self.simulation.particles:
            if other is self:
                continue

            # inverse square law
            displacement = other.position - self.position
            force += displacement / math.hypot(*displacement) ** 3

        return force

    def update(self, dt):
        force = self.calculate_force()
        self.velocity += dt * force / self.mass
        self.position += dt * self.velocity

```


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


## Facade pattern and encapsulation

Wikipedia says of the **facade pattern**
([permalink](https://en.wikipedia.org/w/index.php?title=Facade_pattern&oldid=996427370)):

> Analogous to a facade in architecture, a **facade** is an object that
> serves as a front-facing interface masking more complex underlying or
> structural code. A facade can improve the readability and usability of
> a software library by **masking interaction with more complex
> components behind a single (and often simplified) API**. [...]
>
> Developers often use the facade design pattern when a system is very
> complex or difficult to understand because the system has many
> interdependent classes or because its source code is unavailable. This
> pattern hides the complexities of the larger system and provides a
> simpler interface to the client. It typically involves a single
> wrapper class that contains a set of members required by the client.
> These members access the system on behalf of the facade client and
> hide the implementation details.

An API is a common example of a facade: the client does not need to know
the machinery of the server as it processes a request, and in a
properly-designed API the behaviour of the client code should depend
only on the behaviour of the API functions.

The facade pattern is closely linked to the principle of
**encapsulation**: that a properly-encapsulated object's properties may
be queried or set, but the underlying implementation or storage should
not be exposed. In this way, the implementation of an object may be
freely changed without affecting its usage or behaviour.

Consider for example a class `Point3D`, representing a point in space.
One possibility is to describe the point in terms of its Cartesian
coordinates:
```python
@dataclass
class Point3D:
    x: float
    y: float
    z: float
```

Suppose we want to work out the distance of a point from the origin.
This can be done by defining a function
```python
from math import hypot

def distance_from_origin(point: Point3D) -> float:
    return hypot(point.x, point.y, point.z)
```

However, our user code now depends on the instance variables `.x`, `.y`
and `.z`: both in the definition of `distance_from_origin`, as well as
in a constructor expression such as `Point3D(3, 6, 1)`, where the
arguments will be treated as Cartesians.  What if one day we decide to
represent points in terms of their spherical or cylindrical polar
coordinates instead, or indeed some other curvilinear system?

There are several steps that will make the interface to `Point3D` more
agnostic to its implementation. First, we can provide the distance from
the origin as a `@property`, alongside any other coordinates:
```python
@dataclass
class Point3D:
    ...

    @property
    def r(self):
        """Radial coordinate in sphericals"""
        return hypot(self.x, self.y, self.z)

    @property
    def rho(self):
        """Radial coordinate in cylindricals"""
        return hypot(self.x, self.y)

    @property
    def theta(self): ...

    @property
    def phi(self): ...
```
In this way, instead of `distance_from_origin` we now simply need
`point.r`.

Next, we provide constructor methods for creating `Point3D` objects from
different coordinate systems, *including* from Cartesians.
```python
@dataclass
class Point3D:
    ...

    @classmethod
    def from_cartesians(cls, x, y, z):
        return cls(x=x, y=y, z=z)

    @classmethod
    def from_sphericals(cls, r, theta, phi):
        return cls(
            x=r * sin(theta) * cos(phi),
            y=r * sin(theta) * sin(phi),
            z=r * cos(theta),
        )
```
A call to `Point3D.from_cartesians(3, 6, 1)` is now explicit that those
numbers represent Cartesian coordinates.

Furthermore, we can use
```python
@dataclass(kw_only=True)
class Point3D:
    ...
```
to enforce that the arguments to the default constructor `__init__` must
be specified as keyword arguments, to further reduce ambiguity.


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
