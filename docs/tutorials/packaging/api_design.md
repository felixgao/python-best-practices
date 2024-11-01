!!! Summary

    :white_check_mark: Use a flat namespace for packages/modules.
    
    :white_check_mark: Use primitive-typed functions/methods.

    :white_check_mark: High cohesion & low coupling.
    
    :x: Avoid Low cohension & high coupling.

    :x: Avoid pandas objects as arguments and returns.
    
    :x: Avoid exposing classes from packages/modules.


# API Design

This guide uses the term "API" to refer to the interface of a package, module, 
class or function. It outlines specific recommendations for each type, but general rules apply to all.

**API design is arguably the most important part of writing clean reusable code.**
A well-designed API should be intuitive and straightforward for users, requiring minimal reference to documentation.

This topic is particularly relevant to data science code because API design often becomes an afterthought compared to the actual functionality and data modeling. This leads to code that's often single-use, even when the core functionality is quite general. Additionally, duplicate functionality gets implemented in separate repositories by different developers who find it not worthwhile to adapt hardcoded/specialized logic to their projects.

This problem can also exist within a single project/repository. Modules can be tailored so specifically to a dataset or model architecture that using them outside of that context becomes impossible.


## Goals of a Good API

A well-designed API should exhibit the following properties:

- **Legibility**: Users should understand how to utilize an API immediately. The meaning of arguments and return values should be obvious, requiring minimal documentation.
- **State Simplicity**: Users should readily comprehend the state's lifetime within an API. There should be no hidden caching behavior, allowing users to deallocate state on demand.
- **Interoperability**: Users should immediately understand how to utilize library functions/classes with functions/classes from other libraries, even outside of the given library.


## Example Libraries

- **Python Standard Library**: This library embodies a consistent style and uses patterns that make it very user-friendly.
- **NumPy**: This function-oriented library exposes one primary class and numerous functions. For most `numpy` functions, the first argument is typically a `np.ndarray`, and the output is also a `np.ndarray`. This means that understanding the vast array of functions in the library only necessitates familiarity with one object type.
- **Scikit-learn**: This object-oriented library exposes many model classes. Most `scikit-learn` objects inherit from a small subset of base classes with well-defined methods. Nearly all `scikit-learn` objects have a `fit(X, y)` method and a `predict(X)` method. This consistency allows most scikit-learn objects to be interchangeable within user application code.

The libraries mentioned above are all heavily used within data science code
and they follow design patterns that we can learn from. We can derive a base 
set of guidelines from the patterns we see in these interfaces.


## Design Guidelines 

These guidelines are principles to strive for when designing an API, but not absolute rules. Exceptions exist:


1. [Use a flat namespace for packages/modules.](#namespace)
2. [Use primitive types in function/method interfaces.](#types)
    - [Avoid dictionary-oriented interfaces.](#dictionary-types)
    - [Avoid pandas-oriented interfaces.](#pandas-types)
3. [Avoid variable length keyword arguments (**kwargs)](#kwargs)
4. [Avoid exposing stateful objects. Use mostly functions.](#classes)
5. [Minimize disk access and serialization.](#serialization)


### Use a flat namespace for packages/modules {#namespace}

This makes it so that a developer coming into your code will always know what 
publicly accessible identifiers are available at the library level. This
also avoids having to make multiple imports from the same library and polluting
application code namespace with library objects/modules. Furthermore this 
discourages users from aliasing imports since the library only exposes a single 
namespace.

!!! success "Use"

    ```python
    import my_library

    result = my_library.submodule1.function1()
    obj = my_library.submodule1.Class1()

    result2 = my_library.submodule2.function2()
    obj2 = my_library.submodule2.Class2()
    ```

!!! fail "Avoid"

    ```python
    from my_library.submodule1 import function as function1, Class1
    from my_library.submodule2 import function as function2, Class2

    result = function1()
    obj = Class1()

    result2 = function2()
    obj2 = Class2()
    ```

An example of a library that does this extremely well is [numpy].


### Use primitive types in function/method interfaces {#types}

The goal is to create simple and intuitive function/method interfaces by primarily using primitive data types (e.g., `str`, `int`, `float`, `bool`). This promotes code readability, maintainability, and testability.

Why Primitive Types?
- **Clarity**: Functions with clear input and output types are easier to understand and use.
- **Testability**: Unit tests can be written more easily when dealing with simple data types.
- **Flexibility**: Functions that rely on primitive types are more adaptable to different use cases.

#### When to Consider Complex Data Structures
While primitive types are generally preferred, there are situations where complex data structures like dictionaries or DataFrames might be necessary:

 - **Large Data Set**s: For performance reasons, using NumPy arrays or Pandas DataFrames can be more efficient.
 - **Structured Data**: Dictionaries can be useful for representing key-value pairs, especially when the structure is not fixed.
 - **Domain-Specific Data**: In some cases, using custom data classes might be appropriate to model complex domain-specific concepts.

However, even when using complex data structures, strive to minimize their complexity and expose a simple interface to the user.

#### Best Practices
1. **Prioritize Primitive Types**: Always consider using primitive types as the primary building blocks of your API.
2. **Use Type Hints**: Employ Python's type hinting system to improve code readability and catch potential type errors early.
3. **Minimize Dictionary Usage**: If dictionaries are necessary, consider using namedtuples or custom data classes for structured data.
4.** Optimize Object Usage**: When working with Object, focus on exposing a simple interface that hides the underlying complexity.
5. **Handle Errors Gracefully**: Implement robust error handling mechanisms to prevent unexpected behavior and provide informative error messages.
By following these guidelines, you can create APIs that are both powerful and easy to use.

#### Dictionary-Oriented Interfaces {#dictionary-types}

A common pattern is to use dictionary arguments and return values. The problem 
with this kind of design is it often creates an implicit API which is not 
immediately obvious from the function signature. A function that can be 
understood from inspecting the name and arguments is generally better than one
that requires reading the docstring.


##### Dictionary Arguments

Often times deeply nested structures are required as input 
arguments. These structures are nearly always unnecessary to perform the core 
logic/utility provided by the code. The structural requirements of the 
dictionary cannot be known by inspecting the function arguments. The user of 
the function must look at the documentation or the function body to understand 
how to use it.

!!! fail "Avoid"

    ```python
    def join_first_and_last(parts, sep=' '):
        return parts["first"] + sep + parts["last"]
    
    # Example call
    data = {"first": "hello", "last": "world"}
    result = join_first_and_last(data)
    ```

**Solution**: Force the user to destructure the data prior to passing it in to 
the function. This is a good practice because it limits the scope of what each 
function "knows" about. If the function can be written in a way that the 
context in which is it called is unimportant, this makes the code more 
reusable and easier to test.

!!! success "Use"

    ```python
    def join_first_and_last(first, last, sep=' '):
        return first + sep + last
    
    # Example call
    data = {"first": "hello", "last": "world"}
    result = join_first_and_last(data["first"], data["last"])
    ```

The most common exceptions to using a dictionary as an argument 
is when the dictionary is meant to be iterated over within the function or used
as a lookup table.

!!! attention "Exception"

    ```python
    # Iteration usage
    def max_key_length(dictionary: dict):
        return max(map(len, dictionary.keys()))
    
    # Lookup table usage
    def largest_zip_code_population(zip_codes: list, zip_to_population: dict):
        return max(zip_to_population.get(code, -1) for code in zip_codes)
    ```

##### Dictionary Returns

When dictionaries are used as return values, this causes a 
similar problem to using them as an arguments. The user of the function cannot
determine the structure of the dictionary by inspecting the function signature. 
This also nearly always requires that the user to destructure the return value
in order to use it.

!!! fail "Avoid"

    ```python
    def precision_recall(model, x_test, y_test):
        precision = model.precision(x_test, y_test)
        recall = model.recall(x_test, y_test)
        return {"precision": precision, "recall": recall}
    
    # Example call
    result = precision_recall(...)
    precision = result["precision"]
    recall = result["recall"]
    ```

Use tuples as returns. This avoids forcing the user to 
destructure the return value. Notice that in the above example the dictionary 
keys needed to be duplicated in the calling code in order to access the data. 

!!! success "Use"

    ```python
    def precision_recall(model, x_test, y_test):
        precision = model.precision(x_test, y_test)
        recall = model.recall(x_test, y_test)
        return precision, recall
    
    # Example call
    precision, recall = precision_recall(...)
    ```

The most common exception to using a dictionary as a return type 
is when the  dictionary is meant to be iterated over by the calling code or used
as a lookup table.


!!! attention "Exception"

    ```python
    def get_column_types():
        return {
            'zip_code': str,
            'salaries_and_wages': int,
            'flag_old_or_blind': bool,
        }
    ```


#### Pandas-Oriented Interfaces {#pandas-types}

The worst functions are those that consume DataFrames as arguments and have very 
specific requirements on what columns are present. This is one of the most
common anti-patterns found in data science python code. This creates an implicit 
API that in the worst case (and most common case) requires that the developer 
read the entire function body to understand how to use it. Imagine if you needed
to read the body of every function included in the python standard library in 
order to understand it. We take it for granted that the API is so simple and
easy to understand.


##### Pandas Arguments

This is the *most common* type of function in data science codebases (often 
significantly more complex).

!!! fail "Avoid"

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

!!! success "Use"

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


##### Pandas Class Arguments

A similar problem occurs in object-oriented library design but to a worse 
degree. A common anti-pattern in data science code is to unnecessarily save 
entire datasets as a class member (or within multiple classes). This is 
generally done so that the class can later reference that member variable 
from a method without including it in the method argument list.

!!! fail "Avoid"

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
during the entire lifetime of the object. If the `df` is only used in the 
`train` method then it should not be passed to the constructor.

A second issue here is that the config dictionary argument creates an implicit 
API. The user of the class cannot tell what the contents of the config 
dictionary should be without looking at the implementation of the constructor. 
This is why frameworks like `sklearn` make all of their hyperparameters explicit 
in their constructors. It allows you to be able to tell at the call-site if you 
are configuring the estimator correctly. Imagine if the caller had made a typo 
in a dictionary key for the example above. The mistake would be silently 
ignored unless an explict check was implemented. An explicit check would 
introduce unnecessary complexity when a language feature (arguments) is already
available to handle this.

**Solution**: Colocate the DataFrame with its usage and make the class 
constructor take explicit arguments.

!!! success "Use"

    ```python
    class MyPredictor:
        def __init__(self, n_iterations: int):
            self.n_iterations = n_iterations
        def train(self, X: np.ndarray, y: np.ndarray):
            ...
        def predict(self, X: np.ndarray):
            ...
    ```


### Avoid variable length keyword arguments (**kwargs) {#kwargs}

Variable length keyword arguments should almost never be used because they hide 
what is actually being done with them. Whenever they are used, the user will
almost always have to read the documentation to understand how they are being
used.

!!! fail "Avoid"

    ```python
    def example(arg, **kwargs):
        if 'foo' in kwargs: 
            ...
    ```

The only time when keyword arguments reduce ambiguity is if your library is 
wrapping a call to another library which is highly configurable. Rather than 
replicating all of the keyword arguments in the calling function signature, 
forward the configuration parameters with kwargs.

!!! success "Use"

    ```python
    import foo_library
    def example(arg, **foo_params):
        result = foo_library.example_call(**foo_params)
        ...
    ```

Another potential use-case where keyword arguments are useful is if the 
intention is to iterate over the key-value pairs 
(for example to use as metadata). Because this actually changes the call syntax
of the function, it is generally more explicit to pass this kind of object in
as a dictionary instead.

!!! fail "Avoid"

    ```python
    def example(**tags):
        ...
    ```
    
!!! success "Use"

    ```python
    def example(tags: dict):
        ...
    ```


### Avoid exposing stateful objects {#classes}

Only expose classes when a library/module **must** manage state. Object-oriented
data science code is very useful when complex state cannot be abstracted into
procedures without exposing many different internal state variables to the end
user. However, nearly all object-oriented code can be written in a 
procedure-oriented style. The benefit of procedure-oriented code is that it 
makes the lifetime and usage of in-memory state well-understood.

#### Single Method Classes

The most common object-oriented anti-pattern is when a class has a constructor
and only a single public method. This can almost always be replaced with a 
function call with zero cost.

!!! fail "Avoid"

    ```python
    class Greeter:
        def __init__(self, name):
            self.name = name
    
        def greet(self):
            return f'Hello, {self.name}!'
    ```

!!! success "Use"

    ```python
    def greet(name):
        return f'Hello, {name}!'
    ```


#### Argument Avoidance

Another common anti-pattern is the use of classes to reduce the number of 
arguments on each method call. Rather than scoping arguments to the functions 
they are relevant to, all arguments for all methods are passed to the 
constructor and then 0-argument methods called. This design pattern often occurs 
when developers attempt to define a process with an object.

!!! fail "Avoid"

    ```python
    from sklearn.tree import DecisionTreeRegressor
    
    class MyModel:
        def __init__(self, X_train, y_train, X_test, y_test, hyperparams):
            self.X_train = X_train
            self.y_train = y_train
            self.X_test = X_test
            self.y_test = y_test
            self.hyperparams = hyperparams
            self.model = None
        
        def create_model(self):
            self.model = DecisionTreeRegressor(**self.hyperparams)
    
        def train(self):
            if not self.model:
                self.create_model()
            self.model.fit(self.X_train, self.y_train)
    
        def score(self):
            return self.model.score(self.X_test, self.y_test)
    ```

A common problem with this type of design is that the class methods have a 
required call order that is not explicit in the API. In the example above, this
is remedied by checking if member variables have been set. This complicates the 
logic, adds almost no value, and distracts from the actual logical operations
being performed.


!!! success "Use"

    ```python
    from sklearn.tree import DecisionTreeRegressor
    
    def create_model(hyperparams):
        return DecisionTreeRegressor(**hyperparams)
    
    def train(model, X_train, y_train):
        model.fit(X_train, y_train)
    
    def score(model, X_test, y_test):
        return model.score(X_test, y_test)
    ```


### Minimize disk access and serialization. {#serialization}

It is common for data science code to produce and consume many artifacts during
both training and prediction. This causes many codebases to become saturated 
with disk access to these artifacts and hardcoded paths in every single module. 
In codebases where this is left unchecked, disk access can be nearly everywhere 
and cause huge performance issues. It is common to find a class which has a 
path as a constuctor argument to load some  config or data.

Functions/classes should almost never read from disk to execute the provided 
logic. The logic and the state management should always be separated.

The following example may seem like an egregious use of serialization, but 
these patterns can all be commonly found in code.

!!! fail "Avoid"

    ```python
    import numpy as np
    import yaml
    
    class Encoder:
        def __init__(self, config_filename: str, categories_filename: str):    
            config = yaml.load(config_filename, Loader=yaml.FullLoader)
            self.min_frequency = config.get('min_frequency', 10)
            self.default = config.get('default', -1)
    
            self.categories = None
            if categories_filename is not None:
                self.categories = np.load(categories_filename)
    
        def train(self, training_filename: str):
            values = np.load(training_filename)
            unique, counts = np.unique(values, return_counts=True)
            categories = unique[counts >= self.min_frequency]
            self.categories = categories
        
        def predict(self, test_filename):
            values = np.load(test_filename)
            lookup_table = dict(zip(self.categories, range(len(self.categories))))
            return np.array([lookup_table.get(item, self.default) for item in values])
    
        def save(self, output_filename):
            np.save(output_filename, self.categories)
    ```

The constructor expects two arguments, one for configuration and one for a 
previously saved state. Unless the function body were read, it would be unclear
that the second argument was used to load previously saved state. Next, each
function takes a filename as input instead of an in-memory object which makes 
the object far more difficult to use.

**Solution**: The exposed API should always assume that configurations and
parameters will be in-memory (unless they absolutely cannot be). This vastly
simplifies the usage of the object.

!!! success "Use"

    ```python
    import numpy as np
    
    class Encoder:
        def __init__(self, min_frequency=10, default=-1):    
            self.min_frequency = min_frequency
            self.default = default
            self.lookup_table = None
    
        def train(self, values: np.ndarray):
            unique, counts = np.unique(values, return_counts=True)
            categories = unique[counts >= self.min_frequency]
            self.lookup_table = dict(zip(categories, range(len(categories))))
        
        def predict(self, values):
            return np.array([self.lookup_table.get(item, self.default) for item in values])
    ```

This design requires that the `Encoder` object would be serialized by 
user. This is preferable to implementing custom serialization logic as part of
the class since the user can use common shared serialization facilities 
(e.g. pickle) to manage disk access.

### Cohesion & Coupling

>
> High cohesion is when you have a class that does a well defined job. 
> Low cohesion is when a class does a lot of jobs that does't have much in common.

>
> High coupling is when modules are tightly interconnected via many complex interfaces and information flows.
> Low coupling is when modules are loosely interconnected and isolated from the implementation details of each other.  One module can be changed or replaced without impacting other modules. 

Coupling and cohension goes in pairs like the two faces of a coin.  

Cohension is like an onion -- it requires layers to keep things separate and distinct.  It is not just about the size of your functions but also the relationships with each other. 

Coupling is like a knife -- it cuts through the layers and connects multiple facets. It is not just about the number of objects but also the degree of mutual interdependence.

A good book for this is described in Chapter 10 of [code like a pro] book.



[python standard library]: https://docs.python.org/3/library/
[numpy]: https://docs.scipy.org/doc/numpy/
[sklearn]: https://scikit-learn.org/stable/modules/classes.html
[code like a pro]: https://livebook.manning.com/book/code-like-a-pro/chapter-10
