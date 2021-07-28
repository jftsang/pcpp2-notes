# Metaclasses and Mixins

Comments and corrections to [J. M. F. Tsang](j.m.f.tsang@cantab.net).

---


## Metaclasses

A **metaclass** is a class whose instances are classes. A more descriptive name would be a **class constructor**. They are related to templates in C++ or generics in Java.

The prototypical metaclass in Python is `type`, because every class is a `type`:

```python
Class Spam:
    pass
    
assert isinstance(Spam, type)
```

And a metaclass is (probably) a subclass of `type`.

Like most other topics in object-oriented programming, the purpose of metaclasses is to reduce the amount of repeated code by implementing the functionality of that code in an independent capsule. 

### Example: All Argument Constructors

Consider the following two classes, which might be used in a particle simulation. 
```python
class Species:
    def __init__(self, density, elasticity, friction):
        self.density = density
        self.elasticity = elasticity
        self.friction = friction


class Particle:
    def __init__(
        self, 
        position: Tuple[float, float, float], 
        radius : float, 
        species : Species
    ):
        self.position = position
        self.radius = radius
        self.species = species
        
        @property
        def mass(self):
            return (
                4.0 / 3.0 * math.pi * self.radius ** 3
                * self.species.density
            )
```
Creating members of these classes looks like this:
```python
steel = Species(1, 0.7, 0.1)  # in nondimensional units#
particle = Particle((0, 0, 0), 1.5, steel)
```
Like many other classes, in both cases all that the `__init__` methods do is to copy its keyword arguments into `self`. They could have been written like this:
```python
class Species:
    members = ["density", "elasticity", "friction"]
    
    def __init__(self, **kwargs):
        for key in kwargs:
            if key not in members:
                raise TypeError(f"{key} not in members for this class")
            setattr(self, key, kwargs[key])
            
# simile Particle
```
Wouldn't it be nice if we didn't have to write this out for every class?

One solution is to introduce a metaclass that produces classes with this `__init__` function. Let's call this metaclass `AttributeInitType`, and define it as follows ([Wikipedia](https://en.wikipedia.org/w/index.php?title=Metaclass&oldid=1031439786)):
```python
class AttributeInitType(type):
    def __call__(self, *args, **kwargs):
        """Create a new instance."""

        # First, create the object in the normal default way.
        obj = type.__call__(self, *args)

        # Additionally, set attributes on the new object.
        for name, value in kwargs.items():
            setattr(obj, name, value)

        # Return the new object.
        return obj
```

And now,
```python
class Species(metaclass=AttributeInitType):
    pass
    
class Particle(metaclass=AttributeInitType):
    @property
    def mass(self):
        ...
```


#### Alternative using a class decorator

All that said, I find the following a bit neater. In Java, the Lombok package offers the `@AllArgsConstructor` annotation which provides such a constructor:
```java
@AllArgsConstructor
public class Particle {
    double x, y, z, radius;
    Species species;
    // no need to write your own constructor
    // for assigning these values
}

Particle p = new Particle(...);
```
A similar approach in Python would be to use a [[My favourite decorators|class decorator]] on the class:
```python
@all_args_constructor
class Particle:
    members = {"position", "radius", "species"}
    
    @property
    def mass(self):
        return (
            4.0 / 3.0 * math.pi * self.radius ** 3
            * self.species.density
        )        
```
The decorator could look like this:
```python
def all_args_constructor(cls):
    """A class decorator that 
    def init(instance, **kwargs):
        if set(kwargs.keys()) != set(cls.members):
            raise TypeError("Constructor arguments don't match")
        
        for key in kwargs:
            setattr(instance, key, kwargs[key])

    cls.__init__ = init
    return cls
```
equipping the class with an `__init__` method with nothing more than having a class member called `members`.

**Exercise:** Write a decorator called `@defaults_constructor` that allows the `members` member to be a dictionary that specifies default values if they aren't specified in the initialisation of an instance.