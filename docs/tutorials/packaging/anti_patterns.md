!!! Summary

    :x: Avoid blanket try/except block.

# Anti-Patterns

Below is a list of anti-patterns that we should avoid in our code bases.

## Maintanance

A program is said to be maintainable if it is easy to understand and modify as per the requirement.

### Blanket Try/Except Block

!!! fail "Avoid"

    ```python
    try:
        some_function()
    except Exception as e:
        pass
    ```

!!! fail "Avoid"

    ```python
    try:
        some_function()
    except:
        log()
    ```

Don't catch exceptions unless there is something you can do the fix it.
It is usually a sign of badly designed API that uses exceptions to control flow. 

!!! success "Use"

    ```python
    try:
        some_function(my_input)
    except FixableException as e: # Catch specific exception
        fix_the_problem()
        log.info(f"fixing the problem encountered by {my_input=}")
    ```

!!! success "Use"

    ```python
    try:
        some_function(my_input)
    except NotFixableException as e:
        log.error(f"I can't do anything with {my_input=}")
        raise e # it is also good idea to wrap your own exception `raise MyNewException(e)`
    ```

### Business logic in `__init__.py`

`__init__.py` just like any files, it can contains any legal python code.  The primary use of `__init__.py` is for python to understand the package structure of your code.  A good use of 
`__init__.py` is to use it to expose objects and types that others could use.  Do not have custom logics in this file.

!!! fail "Avoid"

    ```python
    # inside __init__.py
    import my_module

    def some_function(lst: List[int]) -> str:
        return "".join(lst)

    class MyObject:
        def __init__(self):
            ...
        def call(self):
            ...
    ```

!!! success "Use"

    ```python
    # inside __init__.py
    from my_module import myfunc, MyClass
    from my_second_module import myfunc as my_second_func, MyClass as MySecondClass
    from some_other_module import create_singleton

    my_singleton_obj = create_singleton()
    ```


### Avoid wildcard imports (import the world)

During python's import mechanism any code in the module's import will be executed.



!!! fail "Avoid"

    ```python
    from my_module import *
    ```

!!! success "Use"

    ```python
    from my_module import (
        my_func,
        MyClass
    )
    ```


### Not using ContextManager with stateful code

Whenever you have code that is stateful with system resources like openning a file or create a network socket, you should use context manager to handle the context.  

!!! fail "Avoid"

    ```python
    f = open(path)
    for l in f:
        do_something(l)
    f.close()
    ```

!!! success "Use"

    ```python
    with open(path) as f:
        for l in f:
            do_something(l)
    ```

Another important thing to note is that context manager is created for handling potential failures of system resources. Do not abuse it by using the context manager for things that doesn't require entering a context and exiting the context when done. 

### Returning multiple variable types

If a function is suppose to return a given type (e.g. List, Tuple, Dict, MyObject) suddenly returns something else (e.g. `None`)

!!! fail "Avoid"

    ```python
    def get_meaning_of_life(question: str) -> str:
        if question != "The Hitchhiker's Guide to the Galaxy":
            return None
        else:
            return "42"
    ```

!!! fail "Avoid"

    ```python
    def parse_value(try_value: str) -> Union[int, str]:
        try:
            return int(try_value)
        except ValueError as e: 
            pass  # this is also anti-pattern, using exception as flow control

        if not try_value.isalpha():
            return None

        return try_value
    ```

!!! success "Use"

    ```python
    def get_meaning_of_life(question: str) -> str:
        if question != "The Hitchhiker's Guide to the Galaxy":
            raise UnknownQuestionException("Marvin!")
        else:
            return "42"
    ```

### Using single letter or abbeviations in your variable name

variable names should follow the principle of least astonishment.  One-character names should generally be avoided, because they contain little to no information about what they refer to. However, there are a couple of exceptions that make some sense in their given contexts.

!!! fail "Avoid"

    ```python
    passwd = "abc"
    cust = fname = "John"
    comp = "Intuit"
    
    d = {"key": "val"}
    l = [1,2,3]
    t = (1,2)
    ```

!!! success "Use"

    ```python
    password = "abc"
    customer = "John"
    company = "Intuit"

    lookup = {"key", "val"}
    short_ints = [1,2,3]
    temp_slice = (1,2)

    for key, value in lookup:
        if key == "data":
            print(f"{key=}, {value=}"
    ```

### Passing implicity data shapes arounds

Generic containers is great for simple data types but should be avoided to hold nested values.

!!! fail "Avoid"

    ```python
    def process(data:Dict) -> bool:
        data_container = data["my_key"][0][-1]
        pretty_data = prettify(data_container)
        data["my_key"][0][-1]["Data"] = pretty_data
    ```

You should use a dataclass or pydantic datamodel to model the object you want to operate on.

### Access protected members from outside the class

Variables with leading `_` are considered protected members. Access to this variable directly from outside of the class is dangerous and potential to run time errors.

!!! fail "Avoid"

    ```python
    class Rectangle(object):
        def __init__(self, width, height):
            self._width = width
            self._height = height

    rectangle = Rectangle(5, 6)
    calculate(rectangle._width, rectangle._height)
    ```

This is also an example of over-engineering.  If all you need is some data that stores values. you can simple create dataclasses.

!!! success "Use"

    ```python
    @dataclass
    class Rectangle:
        width: Number
        height: Number

        def area(self) -> Number:
            return self.width * self.height
    ```

### Assigning to built-in (reserved) keywards

Python has a number of built-in functions that are always accessible in the interpreter. Unless you have a special reason, you should neither overwrite these functions nor assign a value to a variable that has the same name as a built-in function. Overwriting a built-in might have undesired side effects or can cause runtime errors. Python developers usually use built-ins ‘as-is’. If their behaviour is changed, it can be very tricky to trace back the actual error.


!!! fail "Avoid"

    ```python
    list = [1, 2, 3]
    my_list = list() # Error: TypeError: 'list' object is not callable

    sum = len # reassigned sum to be len operator
    sum(range(10)) # instead of 45, you get 10
    ```

### Python is not Java

Both Python and Java is object oriented programming langauge. However, there is Java way of doing things and Pythonic way of doing things.  Do not translate directly from Java to Python.

!!! fail "Avoid"

    ```python
    class AreaCalculator:

        @staticmethod
        def calc_rectangle(width:Number, height: Number) -> Number:
            return width*height
    ```
the the above example the method `calc_rectangle` can just be a simple function since it does not require any internal states of the `AreaCalculator`.

!!! fail "Avoid"

    ```python
    class Rectangle:

        def set_width(self, width:Number):
            self._width = width
        def set_height(self, height:Number):
            self._width = width
        def get_width(self) -> Number:
            return self._width
        def set_heigh(self) -> Number:
            return self._height
    ```


### Mixing positional and keyword arguments in function. 

prior to Python3.8 mixing positional argument with keyword arguments can lead to confusing results.

!!! fail "Avoid"

    ```python
    def calculate_compound_interest(principle: Number, 
                                    rate: Number, 
                                    terms: Number, 
                                    compounded_monthly:bool=False, 
                                    to_string:bool=False):
        ...
    interest = calculate_compound_interest(1_000_000, 2.5, 10, True, False)
    interest2 = calculate_compound_interest(1_000_000, 2.5, 10, to_string=True, compounded_monthly=True)
    ```
the caller of the function have to be aware of the keyword argument's order because it can be ambigous on which one is first or second.

!!! success "Use"

    ```python
    def calculate_compound_interest(principle: Number, 
                                    rate: Number, 
                                    terms: Number, 
                                     /, *, # Changed to indicate positional arguments ends
                                    compounded_monthly:bool=False, 
                                    to_string:bool=False):
        ...
    # interest = calculate_compound_interest(1_000_000, 2.5, 10, True, False) # this will fail
    interest2 = calculate_compound_interest(1_000_000, 2.5, 10, to_string=True, compounded_monthly=True)
    ```

### Redundant Context 

Do not add unnecessary data to variable names, especially if the context is already clear
!!! fail "Avoid"

    ```python
    class Person:
        def __init__(self, person_first_name, person_last_name, person_age):
            self.person_first_name = person_first_name
            self.person_last_name = person_last_name
            self.person_age_name = person_age_name
    ```

!!! success "Use"

    ```python
    class Person:
        def __init__(self, first_name, last_name, age):
            self.first_name = first_name
            self.last_name = last_name
            self.age_name = age_name
    ```

### Mixing Configs and Constants

Do not mix stuff that can change vs the values that doesn't change in one file.  If your configs is getting longer than one page, you should consider externalize the config into a file. 

!!! fail "Avoid"

    ```python
    # inside config.py
    MY_CONST = 123
    MY_DICT = {
        "my_key": "myvalue",
        ...
        "last_key": "lastvalue"
    }
    ...
    # more stuff
    some_variable = os.enviorn["ENV_VAR_NAME"]
    ```

!!! success "Use"

    ```python
    # inside constant.py
    MY_CONST = 123
    MY_DICT = {
        "my_key": "myvalue",
        ...
        "last_key": "lastvalue"
    }
    ...
    # more stuff

    # inside config.py
    # If you only have a few options here
    some_variable = os.enviorn["ENV_VAR_NAME"]
    ```

!!! success "Use"

    ```python
    # inside constant.py
    MY_CONST = 123
    MY_DICT = {
        "my_key": "myvalue",
        ...
        "last_key": "lastvalue"
    }
    ...
    # more stuff

    # inside config.py
    # if you have business logic or longer than 5 options
    class FeatureIsNoneError(Exception):
        pass

    class TorchModelSetting(BaseModel):
        version: str
        device_count: 

    class InterpolationSetting(BaseModel):
        enable_gpu: Optional[bool]
        model_settings: TorchModelSetting
        interpolation_factor: Optional[conint(gt=2)]
        interpolation_method: Optional[str]
        interpolate_on_integral: Optional[bool]
    
        class Config:
            extra = "forbid"
        
    ```

### Using in to check containment in a (large) list

lack of understanding of the internals usually results in unexpected performance impacts.
Checking if an element is contained in a list using the `in` operator might be slow for large lists.  If you must check this often, consider to change the list to a set or use `bisect`.

!!! fail "Avoid"

    ```python
    list_of_large_items = [...]
    for key in some_other_iterable:
        if key in list_of_large_items:
            do_something()
    ```

!!! success "Use"

    ```python
    list_of_large_items = [...]
    large_items = set(list_of_large_items)
    for key in some_other_relatively_large_iterable:
        if key in large_items:
            do_something()
    ```

if you only need a few checks, it is better to use `bisect`

!!! success "Use"

    ```python
    list_of_a_handful_items = [...]
    list_of_large_items = [...]

    def contains(a, x):
        'Locate the leftmost value exactly equal to x'
        i = bisect_left(a, x)
        if i != len(a) and a[i] == x:
            return True
        return False

    for key in list_of_a_handful_items:
        if contains(list_of_large_items, key):
            do_something()
    ```


### Flattening Arrow Code

Stacked and nested if (conditional) statements make it hard to follow the code logic.
Instead of nesting conditions you can combine them with boolean operators 

!!! fail "Avoid"

    ```python
    user = "Snorlax"
    age = 30
    job = "data scientist"

    if age > 30:
        if user == "Snorlax":
            if job == "data scientist":
                do_science_work()
            else:
                do_work()

    ```

!!! success "Use"

    ```python
    def is_data_scienstist(age:int, user:str, job:str) -> bool:
        ...
    
    if is_data_scienstist():
        do_science_work()
    else:
        do_work()
    ```

if the number of conditions is simple you can use boolean operator to connect

!!! success "Use"

    ```python
    if  age > 30 and user == "Snorlax" and job == "data scientist":
        do_science_work()
    else:
        do_work()
    ```