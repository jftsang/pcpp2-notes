# How to avoid some Python antipatterns
J. M. F. Tsang (j.m.f.tsang@cantab.net)

---

## Data structures
### Dicts, sets and lists
Use a set instead of a list when order doesn't matter, such as with dictionary keys.
```python
# instead of
keys = list(d.keys())

# prefer
keys = set(d.keys())

# don't do these
keys = [*list(d.keys())]
keys = {*list(d.keys())}
```
Sets have the advantage of faster lookup than lists, and also emphasises the unordered nature of the collection.

### Use `.items()` to iterate key-value pairs in a dictionary
```python
# instead of
for key in dic:
    something(key, dic[key])
    and_then(key, dic[key])

# prefer
for key, value in dic.items():
    something(key, value)
    and_then(key, value)
```
### Tuples vs. lists vs. sets
Use tuples when the structure has a definite number of elements that might not be of the same data type. Use lists for collections of arbitrary length whose elements are of the same type. Use sets when order doesn't matter.
```
def f() -> Tuple[int, float, str, Cat]:
    """Returns a collection of things of different types."""
    cat = Cat()
    return 5, 1.0, 'foo', cat


# expanding the returned value
x, y, s, c = f()
```
(Note that all objects in Python are instances of the base type `object`, so technically, *any* list is a collection of homogeneous types.)

### Dictionary order
Don't rely on dictionary ordering.

## Conditionals
### Use guard clauses to avoid excess indentation
Kyle from Web Dev Simplified [completely recommends against `else`](https://www.youtube.com/watch?v=EumXak7TyQ0), in favour of guard clauses. While this is an extreme position, guard clauses do help to avoid indentation (or nested blocks).

In a function:
```python
# instead of 
def f(x):
    if predicate(x):
		do_stuff(x)
		do_more_stuff(x)
		do_yet_more_stuff(x)
		for i in [1, 2, 3]:
			stuff_in_yet_another_layer_of_indentation(i, x)
			more_stuff_in_yet_another_layer_of_indentation(i, x)
	else:
		print('Just a simple message')
		return

# prefer
def f(x):
	if not predicate(x):
		print('Just a simple message')
		return

	# then all this stuff is 
	do_stuff(x)
	do_more_stuff(x)
	do_yet_more_stuff(x)
	for i in [1, 2, 3]:
		stuff_in_yet_another_layer_of_indentation(i, x)
		more_stuff_in_yet_another_layer_of_indentation(i, x)
else:
	print('Just a simple message')
```

In a loop, use `continue` instead of `return`:
```python
# instead of
for x in xs:
	if predicate(x):
		do_stuff(x)
		do_more_stuff(x)
		...
	else:
		print(f'Skipping {x}')

# prefer
for x in xs:
	if not predicate(x):
		print(f'Skipping {x}')
		continue
		
	do_stuff(x)
	do_more_stuff(x)
	...
```

### Prefer `if` over `if not`
If you do want to use an `if: ... else: ...`, then avoid a double negative: 
```python
# instead of
if not predicate(x):
    foo()
else:
	bar()

# prefer
if predicate(x):
	bar()
else:
	foo()
```


## Unnecessary looping
### Prefer comprehensions over loops
```python
# instead of
filtered_squares = []
for x in xs:
    if predicate(x):
	    filtered_squares.append(x ** 2)

# prefer
filtered_squares = [x ** 2 for x in xs if predicate(x)]
```

### Using `map` and `filter`
For large operations, `map` and `filter` can be more memory-efficient than a comprehension, since they produce iterators instead of storing the entire output in memory.
```python
filtered_squares_iter = map(
	 lambda x: x ** 2,
	 filter(predicate, xs)
)
```
You can use the iterable in a `for` loop or a comprehension.

Alternatively, the returned object can be converted into a list (or set):
```python
filtered_squares = list(filtered_squares_iter)
```
but this loses the advantage of memory efficiency.


## Mutability
### Mutable parameters
Avoid unnecessarily passing mutable objects, such as dictionaries, into a function.
```python
# instead of
def set_answer(dic, x):
	dic['answer'] = 3*x

# prefer
def calculate_answer(x)
	return 3*x

dic['answer'] = calculate_answer(x)
```
However, it may be necessary to pass in the dictionary if the answer depends on other values in that dictionary. In that case, still avoid mutating the dictionary inside the function.
```python
# instead of
def set_answer(dic):
	dic['answer'] = dic['some_key'] + dic['some_other_key']

# prefer
def calculate_answer(dic):
	return dic['some_key'] + dic['some_other_key']

dic['answer'] = calculate_answer(dic)
```
This makes the assignment more explicit.

### Default mutable parameters
If your function does need to mutate its input then be careful of using a mutable as its default parameter. Instead, use `None`.
```python
# instead of
def f(xs=[]):
	xs.append(1)

# prefer
def f(xs=None):
    if xs is None:
        xs = []

	xs.append(1)
```


## Exceptions
### Silently passing
Exceptions indicate special cases that *need* to be treated explicitly.
```python
# avoid
try:
    ans = 1 / x
except ZeroDivisionError:
	pass  # no! ans might now be undefined
```


## Comments and work in progress
### Give context for `TODO`s and `FIXME`s
If you are using a bug tracker (Jira, GitHub issues, etc.) then it's a good idea to mark TODOs or FIXMEs to a bug tracker reference.
```python
def f():
	do_stuff()
	# do_unimplemented_stuff()  # TODO PROJ-213
	# do_broken_stuff()  # FIXME PROJ-214
	do_more_stuff()
```

### Use `NotImplementedError`
```python
def f():
	raise NotImplementedError
```
