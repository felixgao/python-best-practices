!!! Summary

    :white_check_mark: Use as much of built-ins as possible.

    :white_check_mark: Use what the language has to offer.

    :x: Avoid blanket try/except block.

    :x: Avoid Repeat Yourself.

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
    def process(data:Dict) -> Dict:
        data_container = data["my_key"][0][-1]
        pretty_data = prettify(data_container)
        data["my_key"][0][-1]["Data"] = pretty_data
        return data
    ```

You should use a dataclass or pydantic datamodel to model the object you want to operate on.

!!! success "Use"

    ```python
    @dataclass
    class LikedLocation:
        places: List[POI]
        likes: List[Number]

        def get_nth_places(self, n:int) -> POI:
            return self.places[n]
        
        def get_first_place(self) -> POI:
            return self.places[0]

    @dataclass
    class POI:
        visited_by: List[Person]

        def get_nth_visited(self, n:int) -> Person:
            return self.visited_by[n]

        def get_last_visited(self) -> Person:
            return self.get_nth_visited(-1)

    @dataclass
    class Person:
        first_name: str
        last_name: str
        date_of_birth: Datetime

    def format_last_visited(locations:LikedLocation) -> `DisplayablePerson`:
        last_visited_by = locations.get_first_place().get_last_visited()
        # assume prettify returns an object of `DisplayablePerson` type
        return prettify(last_visited_by) 

    ```

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


### String manipulation and formatting

String is very versatille type and often you need to convert the data into string.  Not using f-string can lead to a lot of code that is also error prune.

Using f-string it is usually faster than the `format()`  or string subsitution method with `%`.

Rounding numbers to lesser digit

!!! fail "Avoid"

    ```python
    pi = 3.1415926
    pi_3_sifig = str(math.round(pi, 2))
    ```

!!! success "Use"

    ```python
    pi = 3.1415926
    pi_3_sifig = f'{pi:.2f}'
    ```

!!! success "Use"

    ```python
    interest_rate = 0.14159
    interest_rate_percentage = f'{interest_rate:.2%}' # will output '14.16%'
    ```

Create leading digits

!!! fail "Avoid"

    ```python
    month = 1
    month_printable = "0" + str(month) if month < 10 else str(month)
    ```

!!! success "Use"

    ```python
    month = 1
    month_printable = f'{month:02d}'
    ```

adding separator to stringify your number for presentation

!!! fail "Avoid"

    ```python
    revenue = 1000000000 
    revenue_printable = some_function_formatter(revenue, separator=",") # this will return 1,000,000,000
    ```

!!! success "Use"

    ```python
    revenue = 1000000000 
    revenue_printable = f'{revenue:,d}' # you can replace `,` with whatever separate.
    # If you need to pass dynamic separtor, you can do `f"{N: {sep}d}"` where sep is a variable
    ```

If you are using `python 3.8` or newer
You no longer needs to have duplicated logic for constructing text that looks like `name=value`

!!! fail "Avoid"

    ```python
    cost = "$1,000"
    print(f"cost={cost}") # or print("cost={cost}".format(cost=cost))
    # or god forbidden 
    # print("cost =" + cost)
    print("cost=" + (10-len(cost)*" " + cost) # to print 'cost=      $1,000'
    ```

!!! success "Use"

    ```python
    cost = "$1,000"
    print(f"{cost=}") # this is a short hand 
    # if you need to add spaces before the value
    print(f"{cost= :> 10}") # will print, > is right align 'cost=      $1,000'
    ```

Converting datetime or part of it to string

!!! fail "Avoid"

    ```python
    today = datetime.datetime(year=2021, month=8, day=8)
    today_ISO8601_parts = today.strftime("%Y-%m-%d").split("-")
    print(f"today in discouraged format: {today_ISO8601_parts[1]}/{today_ISO8601_parts[2]}/{today_ISO8601_parts[0]}")
    ```

!!! success "Use"

    ```python
    today = datetime.datetime(year=2021, month=8, day=8)
    print(f"today in discouraged format: {today:%m/%d/%Y}, ISO8601 format: {today:%Y-%m-%d}")
    ```

### Not using `defaultdict` or `Counter` instead of `dict`

Python's `dict` is a very versatile container, but it shouldn't be used in certain cases.

- You are trying to count things
- You are manually adding default values


!!! fail "Avoid"

    ```python
    text = "some long text you are trying to count the frequency of words."

    word_count_dict = {}
    for w in text.split(" "):
        if w in word_count_dict:
            word_count_dict[w]+=1
        else:
            word_count_dict[w]=1
    ```

!!! success "Use"

    ```python
    from collections import defaultdict
    word_count_dict = defaultdict(int)
    for w in text.split(" "):
        word_count_dict[w]+=1
    ```

Or even better to use `Counter`

!!! success "Use"

    ```python
    from collections import Counter
    word_counts = Counter(text.split(" "))
    ```

### Using `dict` or mutable type for immutable data

Often `dict` is used as general container of data like config values and etc.  However, if the data is suppose to be immutable, then you really should use the correct immutable types to avoid accidental data overwrite. 

!!! fail "Avoid"

    ```python
    config = {}

    config["service"] = "https://www.intuit.com/prod/service_name"
    config["rate"] = 25
    config["country"] = "US"
    ```

!!! success "Use"

    ```python
    from dataclasses import dataclass

    @dataclass(frozen=True)
    class Config:
        service: str
        rate: int
        country: str
    ```


### Using string instead of enum

String types are a poor choice when the list of possibility is fairly small and finite.   Using string type also could leads to risk of misspelling, escaping exhaustive checks with linters or pattern matching and code duplications.

!!! fail "Avoid"

    ```python
    def train(classifier="tree"):

        if classifier == "tree":
            pass
        elif classifier == "forest":
            pass
        elif classifier == "cnn":
            pass
        else:
            raise ValueError
    ```

!!! success "Use"

    ```python
    from enum import Enum

    class Classifier(Enum):
        TREE = 0
        FOREST = 1
        CNN = 2
        
    def train(classifer: Classifier = Classifier.LBFGS):
        
        if classifer == Classifier.TREE:
            pass
        elif classifer == Classifier.FOREST:
            pass
        elif classifer == Classifier.CNN:
            pass
        else:
            raise ValueError
    ```

with python3.10, you can do pattern matching.

