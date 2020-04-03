!!! Summary

    :white_check_mark: Use a flat namespace
    
    :white_check_mark: Use minimal natively-typed arguments
    
    :x: Avoid exposing classes


# API Design

The goal of any API should be to have the following properties:
- **Legibility**: The user should immediately understand how to use a library 
  function/class. The documentation should be largely unnecessary. How arguments
  function should be obvious. The return type of a function should be obvious.
- **State Simplicity**: The user should immediately understand the state 
  lifetime of a library function/class.
- **Interoperability**: The user should immediately understand how to use 
  library function/class with other functions/classes even outside of the given
  library.

The gold standard for a good python library is the [python standard library]. 
This is a library that follows a consistent style and uses patterns that make it 
extremely easy to use.

Two third party libraries of note that have very easy to understand APIs are:
- [numpy]: Function-oriented library which exposes 1 primary class
- [sklearn]: Object-oriented library which exposes many class

The design of these libraries is almost completely opposite, but they
both manage to make their API easy to understand. By having consistent API rules
in each library it allows users to quickly understand the full scope of the 
library.
 
For `numpy`, each function exposed through the API almost always takes a
`np.ndarray` as the first argument and returns an `np.ndarray` as its output.
This means you are always interacting with the same object type

For `sklearn` each object exposed through the API inherits from a small subset
of base classes with well-defined methods. Almost all `sklearn` objects have a
`fit(X, y)` method and a `predict(X)` method. This makes most `sklearn` objects 
interchangeable within the application code since their APIs are the same.

The 3 libraries mentioned above are all heavily used within data science code
and they each follow different design patterns that we can learn from. We can 
derive a  base set of guidelines from the patterns we see in these APIs.

## Guidelines 

The following guidelines are principals which you should try to adhere to
when designing an API but not absolute strict rules. There are always exceptions
to the following cases:

1. Use a flat namespace.
2. Use primitive types as arguments and returns
3. Avoid variable length keyword arguments (**kwargs)
4. Avoid exposing stateful objects. Use mostly functions.

### Use a flat namespace

This makes it so that a developer coming into your code will always know what 
publicly accessible identifiers are available from at the library level. This
also avoids having to make multiple imports from the same library and polluting
application code namespace with library objects/modules. Furthermore this 
discourages users from aliasing imports since the library only exposes a single 
namespace.

:white_check_mark:
```python
import my_library
result = my_library.my_function()
container = my_library.MyClass()
```
:x:
```python
from my_library.my_submodule import my_function, MyClass
result = my_function()
container = MyClass()

from my_library import my_submodule
result = my_submodule.my_function()
container = my_submodule.MyClass()
```

### Avoid variable length keyword arguments (**kwargs)

Variable length keyword arguments should almost never be used because they hide 
what is actually being done with them. Whenever they are used, the user will
almost always have to read the documentation to understand 

The only time to use them is if your library is wrapping a call to another 
library which is highly configurable. Rather than replicating all of the keyword 
arguments in the calling function signature, forward the configuration parameters 
with kwargs.

Another potential use-case where keyword arguments are useful is if the 
intention is to iterate over the key-value pairs 
(for example to use as metadata). Because this actually changes the call syntax
of the function, it is generally more explicit to pass this kind of object in
as a dictionary instead.


:white_check_mark:
```python
import foo_library
def example(arg, **foo_params):
    result = foo_library.example_call(**foo_params)
    ...
```
:x:
```python
def example(arg, **kwargs):
    if 'foo' in kwargs: 
        ...
```

### Avoid exposing stateful objects

Only expose classes when the library **must** manage state.

[python standard library]: https://docs.python.org/3/library/
[numpy]: https://docs.scipy.org/doc/numpy/
[sklearn]: https://scikit-learn.org/stable/modules/classes.html