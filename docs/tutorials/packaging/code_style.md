!!! Summary

    :white_check_mark: Use [black] code formatting
    
    :white_check_mark: Use [Google-style docstrings]

    :white_check_mark: Use either [napoleon] docstring type annotations or [PEP-484] built-in type annotations 
    
    :white_check_mark: Use [flake8] , [pylint], [pycodestyle] or [yapf] for linting (static code analysis)

    :white_check_mark: (Optional/Strongly Recommended) Use [mypy] or [pyre-check] for type checking


# Code Style

A consistent code style makes code easier to read and understand. The main 
benefit of increased legibility and beautiful code is saving developer time. A 
consistent code style allows anyone to quickly familiarize themselves with a 
codebase even if it is being worked on by many people.

There are many tools/guidelines which can be used to for code style in python 
so a common question is which one to use. The reality is that most of the 
tools/guidelines are fairly good and have generally useful default behavior. 

In practice, for most tools, warnings end up being ignored or specific 
linting options are turned off because they are too restrictive. If a developer 
of a repository disagrees with an option, then they will just remove it 
entirely so that it does not bother them anymore.

When using a set of guidelines which does not have a tool associated with it, 
rules are often forgotten or implemented differently by different developers.

## Goals

Due to the reasons above, the goal of a styling specification would be to 
have the following properties:

- **Automation**: The code style must have a tool. It must have automated 
    formatting so that users do not need to worry about styling the code 
    manually.
- **Minimal Configuration**: Minimize options so that users do not need to make
    decisions about style. More decisions leads to more configuration and more
    inconsistencies in code style.
- **Customizable Rules**: The tool should allow leads to set the type of rules
    going to be enforced and rules can be ignored.

## Example Tools

The following are the most commonly used tools for python code quality and ensure consistency.

### Linters

Linters are usually broken into two types: Logical and Stylistic linters.

Logical Linter:

- Code Errors
- Code with potentially unintended results
- Dangrous code patterns

Stylistic Liners:

- Code not conforming to defined conventions.

Below are some of the popular tools

- [pylint]: A logical and sytlistic liner (more restrictive than [PEP-8])

    The problem with pylint is that there is no automated way to apply the 
    rules to your project. You must run it, look at the errors generated, and
    then update your code manually. This makes it very easy to either ignore 
    the warnings or place unnecessary burden on developers to modify their code 
    until it passes linting. Pylint is opinionated about nearly all aspects of 
    code style including variable naming and number of arguments. This one of 
    the reasons that it is so common to see custom `pylint.rc` files in 
    repositories with specific errors turned off.

    The common complaints against Pylint are that it is slow, too verbose by default, and takes a lot of configuration to get it working the way you want.

- [PyFlake]: A logical liner

    Analyzes programs and detect various errors.  It doesn't complain about style and focus on logical code issues and potential errors.  Therefore, it runs faster than [pylint]. 

- [pycodestyle]: A code stylistic checker formally known as the official [PEP-8] style.

    pycodestyle is a tool to check your Python code against some of the style 
    conventions in [PEP-8].  
    
- [yapf]: An automated code formatter that follows [PEP-8] style.

    Unlike [autopep8] this does not not only remove linting errors, but rather
    reformats all code in an opinionated style (which is [PEP-8] compliant). 
    Like the previous code stylers, this allows for a huge amount of 
    configuration and even allows you to swap out code style entirely.

### Formatters

Formatters will format the actual python file based on rules.

- [black] An automated code formatter with no configuration options.

    A huge benefit of black is that unlike the previously mentioned tools this 
    does not have formatting options. Black does not strictly follow 
    [PEP-8] guidelines but is generally compliant.  It is also important to 
    understand that [black] is only an opinionated `formatter`, it does not
    check all of the styles problem that other `linters` do. 

- [isort] Format imports order.

    Sort imports alphabetically, and automatically separated into sections and by type.

In a very controlled development environment, each may be useful tools by just
enabling the default behavior and forbidding custom options.

## Recommendation

With any styling recommendation it is recommended to pair linters with formatters.

The [black] library is a new code formatter that closely follows the principles 
of the golang formatter [gofmt]. It only enforces the formatting of the code 
like gofmt, black is opinionated and does not allow you to choose how to format 
your code. The correctness of the code is checked upon formatting to ensure that 
no mistake was made when updating the style.

The only caveat to mention is that black is in beta and will not be perfect 
until its official release.

The [pylint] is one of the most popular linting library and well supported. However, 
if you want integration with `pyproject.toml` you can use [flake8] which is a combination of tools ([PyFlakes], [pycodestyle] and [McCabe]). 

We should also use [mypy] to check the type hinting is correct. 


## Style Guides

Beyond automated checks and warnings for code, style guides provide good 
recommendations for how you should write code.

Three style guides in particular provide almost universally useful advice:

- [PEP-8]: The official python style guide provides the basis for almost all 
  formatters and other style guides. This provides the broadest and most 
  foundational rules for writing legible python code.  
- [Google Style Guide]: This has a lot of useful recommendations, however many of 
  their guidelines were chosen to maximize compatibility between python 2 and 
  python 3. Since python 2 is officially deprecated, all of the compatibility
  guidelines should be ignored. For these reasons the main
- [Hitchhiker's Guide to Python]: This provides its own style guide which
  is universally regarded as good practice.

Though none of these style guides can programmatically format code for you, they
are essential reference for how to write clean, simple python code.

## Type Annotations

Type annotations of some kind are recommended but can be implemented in 
different ways. Some form of type annotation is always recommended so that
code usage is less ambiguous.

The recommended strategies for annotating the types of your code:

### [PEP-484] Style type annotations

This places the type annotation directly in code. 

Pros:

- Code can be checked using automated code checkers (See: [mypy] & [pyre-check])

Cons:

- Tends to make function declarations verbose
- Can be difficult to document data structures (Requires importing [typing])

```python
def subtract(minuend: int, subtrahend: int) -> int:
    """
    Subtract the subtrahend from the minuend.
    
    Args:
        minuend: The basis to subtract from.
        subtrahend: The value to subtract.
    Result:
        difference: The difference between the numbers.
    """
    return minuend + subtrahend
```


## Variable Naming

In the style guides mentioned above, the naming conventions are primarily
concerned with the case used (snake_case vs CamelCase, etc.) However, this
section is in regards to the actual words which should be used in a name.

Naming is one of the hardest problems in programming. Good naming can 
drastically increase the readability of your code. To name things well, 
variables should have the following properties:

- Descriptive
- Unambiguous
- Should not contain the type. This is what type annotations are for
- Should be short if possible. Long names make code more difficult to read

Abbrivations for names is ok if it is well-known but should be reframed.
ie. n, sz, cnt, idx, dt, ts, env, cfg, ctx etc. 

### Dictionaries

Dictionaries should have information about both the **key and value** in 
the name. 

Acceptable Forms:

- `{singular-key}_{plural-values}` *(Preferred) Somewhat ambiguous, but succinct*
- `{key}_to_{value}` *Somewhat verbose*
- `{value}_by_{key}` *Somewhat verbose, less direct than `{key}_to_{value}`*
- `{key}_{value}_lookup` *Somewhat verbose, but describes how it should be used*

Each of the forms can be more appropriate in different scenarios where the 
meaning is more clear in context.

!!! success "Use"

    ```python
    word_lengths = {"hello": 5, "world": 5}
    index_to_word = {1: "hello", 2: "world"}
    word_by_index = {1: "hello", 2: "world"}
    index_word_lookup = {1: "hello", 2: "world"}
    ```

!!! fail "Avoid"

    ```python
    words = {1: "hello", 2: "world"}        # Ambiguous, No key information
    word_lookup = {1: "hello", 2: "world"}  # No key information
    word_dict = {1: "hello", 2: "world"}    # Type included in name
    ```

In some cases there are well-understood mappings that are meant to be iterated
over. In these cases it makes sense to just use the pluralized version of the 
word and name the variable after the contents. Anything that is meant to be 
iterated over should be pluralized.

!!! attention "Exception"

    ```python
    headers = {"Content-Type": "application/json"}
    cookies = {"tz": "America/Los_Angeles"}
    hyperparameters = {"min_samples_leaf": 50}
    gunicorn_options = {"workers": 8} # Gunicorn uses `options`
    ```
    
!!! fail "Avoid"

    ```python
    header_name_to_value = {"Content-Type": "application/json"}  # Too verbose
    cookie_name_to_value = {"tz": "America/Los_Angeles"}         # Too verbose
    hyperparameter_name_to_value = {"min_samples_leaf": 50}      # Too verbose
    ```

In other cases, there are libraries that have predefined names for their 
arguments that do not follow the conventions above. In this case it is 
acceptable to follow their conventions when interacting with their code. This
makes the code less ambiguous because the library name and the application-code 
variable name match.

!!! attention "Exception"

    ```python
    feed_dict = {"name": tensor} # TensorFlow uses this name so it is acceptable
    ```

### Lists/Series/Arrays/Sets

Collections (non-dictionary) should always be the plural version of whatever is 
contained within. In the case where the value type is ambiguous, try to name
the collection so it is possible to determine what is contained within.

!!! success "Use"

    ```python
    zip_codes = [92127, 12345]
    names = {"Johnny", "Lisa", "Mark", "Denny"}
    column_names = ["zip_code", "wages"] # `names` suffix indicates string value
    ``
    
!!! fail "Avoid"

    ```python
    zip_list = [92127, 12345]        # Do not include type
    list = [92127, 12345]            # Shadows built-in list, Ambiguous
    items = [92127, 12345]           # Ambiguous
    columns = ["zip_code", "wages"]  # Value type is ambiguous
    ```

### Integers/Floats

When possible, number names should indicate the how the value should be 
used. Contrary to the rules regarding collections, there are a many 
well-understood values that indicate a number which should be unambiguous. 

!!! success "Use"

    ```python
    limit = 10
    threshold = 0.7
    size = 10       # Potentially ambiguous
    index = 10      # Potentially ambiguous
    count = 10      # Potentially ambiguous
    n_items = 10    # `n` prefix always indicates a number, preferred `num_items`
    min_items = 10  # `min` prefix always indicates a number
    max_items = 10  # `max` prefix always indicates a number
    buy_price = 10  # Domain specific words always indicate a number 
    ```

!!! fail "Avoid"

    ```python
    items = 10      # Ambiguous, could indicate a collection
    n = 10          # Not Descriptive
    n_value = 10    # `value` suffix is not descriptive
    n_quantity = 10 # `quantity` suffix is not descriptive
    ```

### Strings

When possible, string names should indicate how the value should be used. The
naming conventions of strings are similar to the conventions of numbers. There 
are many well-understood names that could indicate a string type.

Common Forms:

- `{descriptor}_key` *May indicate lookup key*
- `{descriptor}_name` *May indicate lookup key*

!!! success "Use"

    ```python
    environment_name = 'cdev'
    environment_key = 'cdev'
    environment = 'cdev'      # Well-known string value, Potentially ambiguous
    env = 'cdev'              # Well-known abbreviation, Potentially ambiguous
    ```

!!! fail "Avoid"

    ```python
    value = 'cdev'  # Ambiguous
    config = 'cdev' # Ambiguous could indicate an object
    ```

### Classes

Class instances should always be named after the class itself. For the purposes
of describing the forms which a variable should be named the following class
name will be used `DescriptionNoun`.

Acceptable Forms:

- `{description}_{noun}` *(Preferred) CamelCase to snake_case conversion*
- `{noun}` *More succinct, potentially ambiguous*

!!! success "Use"

    ```python
    class ExampleContext:
        pass
    
    example_context = ExampleContext()
    context = ExampleContext()
    ```

### Functions

Function should always be named in lower case and separated with underscore (`_`). The words that you use to name your function should clearly describe the function’s intent (what the function does).  All functions should clearly indicates the input and output variable types.
When returning from function avoid generic container types like `List` or `Dict`, use additional
hints if you must return such types, ie. `List[int]` or `Dict[str, int]`.  

Acceptable Forms:

- `def {verb}_{intent}({nouns}:T) -> U: ` *(Preferred)*
- `def {verb}({noun}:T}) -> U: ` *More succinct, potentially ambiguous*

!!! success "Use"

    ```python
    def sum(iterable: Iterable) -> Number:
    ```

    ```python
    def remove_underscore(s: str) -> str:
    ```

!!! fail "Avoid"

    ```python
    def foo(a):
    ```

    ```python
    def process(parameters:Dict) -> Dict:
    ```

    ```python
    def calculate(results:Dict[Any, Dict]) -> Dict:
    ```


[yapf]: https://github.com/google/yapf/
[Google-style docstrings]: http://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings
[Hitchhiker's Guide to Python]: https://docs.python-guide.org/writing/style/
[PEP-8]: https://www.python.org/dev/peps/pep-0008/
[pylint]: https://pypi.org/project/pylint/
[flake8]: https://flake8.pycqa.org/en/latest/
[PyFlakes]: https://github.com/PyCQA/pyflakes
[pycodestyle]: https://github.com/PyCQA/pycodestyle
[autopep8]: https://github.com/hhatto/autopep8
[PEP-484]: https://www.python.org/dev/peps/pep-0484/
[mypy]: http://mypy-lang.org/
[pyre-check]: https://pyre-check.org/
[typing]: https://docs.python.org/3/library/typing.html
[napoleon]: https://sphinxcontrib-napoleon.readthedocs.io/en/latest/#type-annotations
[Google Style Guide]: http://google.github.io/styleguide/pyguide.html
[black]: https://github.com/psf/black
[isort]: https://github.com/timothycrosley/isort
[gofmt]: https://golang.org/cmd/gofmt/
[McCabe]: https://github.com/PyCQA/mccabe
