!!! Summary

    :white_check_mark: Use [black] code formatting
    
    :white_check_mark: Use [Google-style docstrings]

    :white_check_mark: Use either [napoleon] docstring type annotations or [PEP-484] built-in type annotations 

    :white_check_mark: (Optional) Use [mypy] or [pyre-check] for type checking
    
    :x: Avoid [pylint], [pep8], and [yapf]


# Code Style

A consistent code style makes code easier to read and understand. If increased
legibility and beautiful code are not enough reason to embrace a consistent 
style, then time is

There are many tools which can be used to check code style in python 
(e.g. [pylint], [pep8], [yapf], etc.) so a common question is which one to use. 
The reality is that most of these tools are fairly good and have generally 
useful default behavior. 

However, in practice, warnings end up being ignored or specific linting options 
are turned off because they are too restrictive. If the developer of a 
repository disagrees with an option, then they will just remove it entirely so 
that it does not bother them anymore.

The best code styling tool/specification would have the following properties:

- **Automation**: Automated formatting to remove styling burden from users.
- **Minimal Configuration**: Minimize options to reduce decisions about style.

The problem with [pylint] is that there is no automated way to apply the rules
to your project. This makes it very easy to either ignore the warnings or place
unnecessary burden on developers to modify their code until it passes linting.

In contrast, a tool like [pep8] also provides automation through [autopep8] 
which will automatically reformat your code rather than just warning you. 
However, even with automation this still allows users to ignore specific
warnings and set their own styling.

Similarly, [yapf] is an automated code formatter, with optional configurations.

In a very controlled development environment, these may be useful tools by just
enabling the default behavior and forbidding custom options.

## Black

The library [black] is a new code formatter that closely follows the principles 
of the golang formatter [gofmt]. Unlike the aforementioned tools, it fulfills
both requirements of having automation and zero configuration. Like gofmt, black
is opinionated and does not allow you to choose how to format your code. The
correctness of the code is checked upon formatting to ensure that no mistake was
made when updating the style.

The only caveat to mention is that black is in beta and will not be perfect 
until its official release.


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
different ways.

There are good strategies for annotating the types of your code:

### [PEP-484] Style type annotations:

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

### Restructured Text type annotations:

Pros:

- Backwards compatible with python 2
- Type-checked by modern interpreters

Cons:

- Code cannot be checked using automated code checkers

```python
def subtract(minuend, subtrahend):
    """
    Subtract the subtrahend from the minuend.
    
    Args:
        minuend (int): The basis to subtract from.
        subtrahend (int): The value to subtract.
    Result:
        difference (int): The difference between the numbers.
    """
    return minuend + subtrahend
```


[yapf]: https://github.com/google/yapf/
[Google-style docstrings]: http://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings
[Hitchhiker's Guide to Python]: https://docs.python-guide.org/writing/style/
[PEP-8]: https://www.python.org/dev/peps/pep-0008/
[pylint]: https://pypi.org/project/pylint/
[pep8]: https://pep8.readthedocs.io/en/latest/
[autopep8]: https://github.com/hhatto/autopep8
[PEP-484]: https://www.python.org/dev/peps/pep-0484/
[mypy]: http://mypy-lang.org/
[pyre-check]: https://pyre-check.org/
[typing]: https://docs.python.org/3/library/typing.html
[napoleon]: https://sphinxcontrib-napoleon.readthedocs.io/en/latest/#type-annotations
[Google Style Guide]: http://google.github.io/styleguide/pyguide.html
[black]: https://github.com/psf/black
[gofmt]: https://golang.org/cmd/gofmt/