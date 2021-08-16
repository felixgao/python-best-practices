!!! Summary

    :white_check_mark: Use [tox] & [pytest] for testing

    :white_check_mark: Use [pytest-mock] or [unittest.mock] for mocking

# Testing

Use Fixtures in `conftest.py` to setup your test. 
Use parameterized tests to test different input use cases.

Generate test coverage report
`pytest tests --cov=${package} --cov-report=term --cov-report xml`

# Survey Of Methods



[tox]: https://tox.readthedocs.io/
[pytest]: https://docs.pytest.org/en/latest/
[pytest-mock]: https://github.com/pytest-dev/pytest-mock/#spy
[unittest.mock]: https://docs.python.org/3/library/unittest.mock.html
