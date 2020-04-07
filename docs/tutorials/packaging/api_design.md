:warning: UNDER CONSTRUCTION :warning:

!!! Summary

    :white_check_mark: Use a flat namespace
    
    :white_check_mark: Use minimal natively-typed arguments
    
    :x: Avoid pandas objects as arguments and returns
    
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
2. Use primitive types as arguments and returns.
    - Avoid dictionaries as arguments and returns.
    - Avoid pandas as arguments and returns.
3. Avoid variable length keyword arguments (**kwargs)
4. Avoid exposing stateful objects. Use mostly functions.
5. Minimize disk access and serialization.

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

### Use primitive types as arguments and returns

To model ourselves after the standard library, attempt to always use functions
which consume native types on the arguments and as return types. The common case
where this is not done is in library oriented code (i.e `numpy`, `pandas`, 
`sklearn`, `tensorflow`, `pytorch`). This is a huge problem in data science
codebases because it becomes difficult to determine what types are expected in
each function. Furthermore, to unit-test a function sometimes requires a
DataFrame as an input. 


#### Dictionary-Oriented Code

A common pattern is to use dictionary arguments and return values. The problem 
with this kind of design is it often creates an implicit API which is not 
immediately obvious from the function signature. A function that can be 
understood from inspecting the name and arguments is generally better than one
that requires reading the docstring.

**Example 1**: Often times deeply nested structures are required as input 
parameters. These structures are often unnecessary to perform the core 
logic/utility provided by the code. This causes the input parameter to have a 
structural requirement that cannot be known by inspecting the function 
arguments. The user of the function must look at the documentation or the 
function body to understand how to use it.

:x:
```python
def join_first_and_last(parts, sep=' '):
    return parts["first"] + sep + parts["last"]

# Example call
data = {"first": "hello", "last": "world"}
result = join_first_and_last(data)
```

**Solution**: Force the user to destructure the data prior to passing it in to 
the function. This is usually a good practice to limit the scope of what each 
function "knows" about. If the function can be written in a way that the 
context in which is it called is unimportant, this  makes the code more 
reusable and generally easier to test.

:white_check_mark:
```python
def join_first_and_last(first, last, sep=' '):
    return first + sep + last

# Example call
data = {"first": "hello", "last": "world"}
result = join_first_and_last(data["first"], data["last"])
```

**Example 2**: When dictionaries are used as return values, this causes a 
similar problem to using them as an arguments. The user of the function cannot
determine the structure of the dictionary by inspecting the signature. This
has the second effect of nearly always requiring the user to destructure the 
return value.

:x:
```python
def truth_values():
    return {"true": True, "false": False}

# Example call
result = truth_values()
true = result["true"]
false = result["false"]
```

**Solution**: Use tuples as returns. This avoids forcing the user to 
destructure the return value. Notice that in the above example the dictionary 
keys needed to be duplicated in the calling code in order to access the data. 

:white_check_mark:
```python
def truth_values():
    return True, False

# Example call
true, false = truth_values()
```

#### Pandas-Oriented Code

The worst functions are those that consume DataFrames as arguments and have very 
specific requirements on what columns are present. This is one of the most
common anti-patterns found in data science python code. This creates an implicit 
API that in the worst case (and most common case) requires that the developer 
read the entire function body to understand how to use it. Imagine if you needed
to read the body of every function included in the python standard library in 
order to understand it. We take it for granted that the API is so simple and
easy to understand.

**Example 1**: This is the *most common* type of function in data science 
codebases (often significantly more complex).

:x:
```python
def typecast_zip_codes(df: pd.DataFrame):
    df['zip'] = df['zip'].astype(int)

# Example call
typecast_zip_codes(df)
```

It is a function designed for a specific dataset that assumes that the calling 
code has a DataFrame with a specific column. It is unclear from the signature
that the DataFrame is modified in-place. It is unclear from the signature if
there is a return that should be used.

**Solution**: Make the API oriented around a series/array and to leave the 
deconstruction of the DataFrame to the calling code.

:white_check_mark:

```python
def typecast_zip_codes(array: np.ndarray):
    return array.astype(int)

# Example call
df['zip'] = typecast_zip_codes(df['zip'].values)
```

The calling code is burdened with the responsibilty of deconstructing/modifying
the DataFrame but this is a *good thing*. This allows the calling code to fully
understand the side-effects of the call without having to look at the 
implementation. This is easier to test and much more maintainable. If this kind 
of API is used throughout the codebase, then there will be very few places that 
have assumptions about DataFrame structure. 

**Example 2**: A similar problem occurs in object oriented problem but to a
worse degree.

:x:

```python
class MyPredictor:
    def __init__(self, df: pd.DataFrame, config: dict):
        self.train_df = df
        self.iterations = config.get('iterations', 0)
    def train(self):
        ...
    def predict(self):
        ...
```

In this code it is confusing what the lifetime of the `df` argument is and what
it is even used for. An object should only track state that must be present 
during the entire lifetime of the object.

The second issue is that the config dictionary argument creates an implicit API.
The user of the class cannot tell what the contents of the config dictionary
should be without looking at the implementation of the constructor. This is why
frameworks like `sklearn` make all of their hyperparameters explicit in their 
constructors. It allows you to be able to tell at the call-site if you are
configuring the estimator correctly. Imagine if the caller had made a typo in
a dictionary key for the example above. The mistake would be silently ignored.

**Solution**: Colocate the DataFrame with its usage and make the class 
constructor take explicit arguments.

:white_check_mark:

```python
class MyPredictor:
    def __init__(self, n_iterations: int):
        self.n_iterations = n_iterations
    def train(self, X, y):
        ...
    def predict(self, X):
        ...
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