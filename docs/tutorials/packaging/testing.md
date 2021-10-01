!!! Summary

    :white_check_mark: Use [tox] or [nox] for automation.

    :white_check_mark: Use [pytest] for testing.

    :white_check_mark: Use [pytest-mock] for mocking.

    :white_check_mark: Use pytest markers to limit test set.

    :white_check_mark: Use [mutmut] for mutation testing.

    :white_check_mark: Use `# pragma: no cover` for lines doesn't need coverage. ie. config lines.

# Testing

- Use Fixtures in `conftest.py` to setup your test.  It should be the basic building blocks.
- Use parameterized tests to test different input use cases.
- Generate test coverage report
 `pytest tests --cov=${package} --cov-report=term --cov-report xml`
- Prefer mocker over mock.
- Ensure a good code coverage.  Preferred to have over 80% coverage.  
- Mutation test can help you figure out if your unit test is good enough.
- Pytest have vast amount of plug-ins to help you make your tests better.

## Fixtures

Fixtures are how test setups (other helpers) are shared between tests. 

- Can build on top of each other to model complext functionality.
- Customize functionality by overriding other fixtures .
- Can be parametrized.

### Yield Fixture aka. Context Fixture

This is a fixture that is created using the yield statement.

```python
@pytest.fixture
def first():
    print("Set up first fixture")
    yield
    print("Clean up first fixture")

@pytest.fixture
def second(first):
    print("Set up second fixture")
    yield
    print("Clean up second fixture")

def test_context_fixture_order(second):
    print("In the test")
    assert False
```

The code above the `yield` is executed as setup for the fixture, while the code after the yield is executed as clean-up.   The value yielded is the fixture value received by the user.  Just like all Context-Managers, every call pushes the context on the stack, it follows the `LIFO` order.  

### Fixture resolution

Pytest uses an in-memory DAG (Directed Acyclic Graph) to figure out what is the order of the fixture a test needs.  Each fixture that is required for execution is run `once`;  The value of the fixture is stored and use to compute the other dependent fixture.  

### Test collection

During test collection, every test module, test class, test function in the [test-discovery] is picked up.  In parallel, every fixture also picked up by inspecting the `conftest.py` files as well as test modules.  If you ever encountered a program during your test and zero test cases ran, it is usually some problem with this phase.  Check your fixture and paramterized code to see if there is any mistakes there.

### Execution

When fixture code is executed, it follows the following rules.

- Session scoped fixtures are executed if they have not already been executed in this test run.  Otherwise, the results of previous execution are used.
- Module scoped fixtures are executed if they have not already been executed as part of this test module in this test run.  Otherwise, the results of the previous execution are used.
- Class scoped fixture are executed if they have not already been executed as part of this class in the test run. Otherwise, the results of previous execution are used.
- Function scoped fixtures are executed.

Once all the fixtures are evaluated, the test function is called with the values for the fixtures filled in. 

## Patterns

The following patterns should be considered when developing tests.


### Fixtures

You should places all your fixtures in various scopes in the correct locations.
session scoped fixtures should be located in `conftest.py` at the package or root level.
class scoped and module scoped fixture should be residing the the `conftest.py` at the same level as your module is defined.
function scoped fixture should be in the same file as the actual test cases. 


### Prefer mocker over mock

Use mocker fixture instead of using mock directly.  

- Eliminate flaky test due to "mock leak" when a test does not reset a patch.
- Reduce boilerplate code
  
```python
# inside of test_mocker.py
def test_fn():
    return 42


class TestMockerFixtureSuite(object):
    """
    Only use a class when you want to organize tests into a suite
    """

    @pytest.mark.xfail(strict=True, msg="We want this test to fail.")
    def test_mocker(self, mocker):
        mocker.patch("test_mocker.test_fn", return_value=84)
        assert another_test_fn() == 84
        assert False

    def test_mocker_follow_up(self):
        assert test_fn() == 42    

    @pytest.fixture
    def mock_fn(self, mocker):
        return mocker.patch("test_mocker.test_fn", return_value=84)

    def test_mocker_with_fixture(self, mock_fn): # notice this function depends on `mock_fn` to override fixture value
        assert another_test_fn() == 84
```


### Parameterize the same behavior

Parameterization allows us to asserting the same behavior with various inputs and expected outputs.  Make separate tests for distinct behaviors.  Use ids to describe individual test cases.

- Copy-pasting code in multiple tests increase boilerplate - use parametrize to reduce this.
- Never loop over test cases inside a test
- Parameterize heterogenous behaviors can lead to complex branching codes and bugs.

```python
# util.py
def divide(a, b):
    return a / b
```

!!! fail "Avoid"

    ```python
    @pytest.mark.parametrize("a, b, expected, is_error", [
        (1, 1, 1, False),
        (42, 1, 42, False),
        (84, 2, 42, False),
        (42, "b", TypeError, True),
        ("a", 42, TypeError, True),
        (42, 0, ZeroDivisionError, True),
    ])
    def test_divide_antipattern(a, b, expected, is_error):
        if is_error:
            with pytest.raises(expected):
                divide(a, b)
        else:
            assert divide(a, b) == expected
    ```

!!! success "Use"

    ```python
    @pytest.mark.parametrize("a, b, expected", [
        (1, 1, 1),
        (42, 1, 42),
        (84, 2, 42),
    ])
    def test_divide_ok(a, b, expected):
        assert divide(a, b) == expected


    @pytest.mark.parametrize("a, b, expected", [
        (42, "b", TypeError),
        ("a", 42, TypeError),
        (42, 0, ZeroDivisionError),
    ])
    def test_divide_error(a, b, expected):
        with pytest.raises(expected):
            divide(a, b)
    ```


### Don't modify fixture values in other fixtures

Pytest test cases are usually executed in parallel but fixtures are usually executed only once.  When multiple fixtures may depend on the same upstream fixture. If any one of these modifies the upstream fixture’s value, all others will also see the modified value; this will lead to unexpected behavior.

If you must override certain values of the parent fixtures, you should make a deepcopy of the data. 

!!! fail "Avoid"

    ```python
    @pytest.fixture
    def engineer():
        return {
            "name": "Alex",
            "team": "Intuit-AI",
        }

    @pytest.fixture
    def ds(engineer):
        engineer["name"] = "Joy"
        return engineer

    def test_antipattern(engineer, ds):
        assert engineer == {"name": "Alex", "team": "Intuit-AI"}
        assert ds == {"name": "Joy", "team": "Intuit-AI"}
    ```
In the above case, since ds modified the value, all of them will have the name `Joy` instead.

!!! success "Use"

    ```python
    @pytest.fixture
    def engineer():
        return {
            "name": "Alex",
            "team": "Intuit-AI",
        }

    @pytest.fixture
    def ds(engineer):
        joy = deepcopy(engineer)  # Create a deepcopy of the object
        joy["name"] = "Joy"
        return joy

    def test_pattern(engineer, ds):
        assert engineer == {"name": "Alex", "team": "Intuit-AI"}
        assert ds == {"name": "Joy", "team": "Intuit-AI"}
    ```


### Prefer response (or replay framework) over mocking outbound HTTP requests

Never manually create Response objects for tests; instead use the [responses] library (if using request library) to define what the expected raw API response is. If using any other http library, you can use [HTTPretty] instead or [respx] if needs [HTTPX] support.


### Prefer tmpdir over static locations

Sometimes in testing you need a directory or files that you can work with to test out your code.  Don't create files in static or predefined locations on your filesystem.  You should use the tmpdir fixture and create the fiels on-the-fly to test.

```python
    # util.py
    def process_file(fp:IO) -> List[int]:
        """Toy function that returns an array of line lengths."""
        return [len(l.strip()) for l in fp.readlines()]
```

!!! fail "Avoid"

    ```python
    @pytest.mark.parametrize("filename, expected", [
        ("first.txt", [3, 3, 3]),
        ("second.txt", [5, 5]),
    ])
    def test_antipattern(filename, expected):
        with open("resources/" + filename) as fp:
            assert process_file(fp) == expected
    ```

!!! success "Use"

    ```python
    @pytest.mark.parametrize("contents, expected", [
        ("foo\nbar\nbaz", [3, 3, 3]),
        ("hello\nworld", [5, 5]),
    ])
    def test_pattern(tmpdir, contents, expected): # tmpdir is build-in to global pytest fixture
        tmp_file = tmpdir.join("testfile.txt")
        tmp_file.write(contents)
        with tmp_file.open() as fp:
            assert process_file(fp) == expected
    ```

### Plugins

- ***pytest-xdist***: run tests in parallel.
- ***pytest-randomly***: run tests randomly (useful to catch weird bugs that runs sequentially).
- ***pytest-sugar***: shows failures and errors instantly and shows a progress bar.
- ***pytest-icdiff***: better diff when asserting error happens. (also pytest-clarity)
- ***pytest-html***: get a html base report of the test.
- ***pytest-instafail***: shows failures and errors instantly instead of waiting until the end of test session.
- ***pytest-timeout***: terminate tests after a certain timeout.
- ***pytest-parallel***: for parallel and concurrent testing.
- ***pytest-picked***: Run the tests related to the unstaged files or the current branch (according to Git).
- ***pytest-benchmark***: fixture for benchmarking code.
- ***pytest-cov***: Code coverage.  (MUST HAVE!)
- ***pytest-lazy-fixture***: allows you to get values of `fixture` from `paratmertized` method.
- ***pytest-freezegun***: Freeze time! 
- ***pytest-leaks***: Find resource leaks.
- ***pytest-deadfixtures***: find out fixtures that are not used or duplicated.
- ***pytest-responses***: fixture for `requests` library mocking.




[tox]: https://tox.readthedocs.io/
[nox]: https://nox.thea.codes/en/stable/
[pytest]: https://docs.pytest.org/en/latest/
[pytest-mock]: https://github.com/pytest-dev/pytest-mock/#spy
[unittest.mock]: https://docs.python.org/3/library/unittest.mock.html
[test-discovery]: https://docs.pytest.org/en/latest/explanation/goodpractices.html#conventions-for-python-test-discovery
[responses]: https://github.com/getsentry/responses
[HTTPretty]: https://github.com/gabrielfalcao/HTTPretty
[respx]: https://github.com/lundberg/respx
[HTTPX]: https://www.python-httpx.org/
[mutmut]: https://pypi.org/project/mutmut/
