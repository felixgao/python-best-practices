!!! Summary

    :white_check_mark: Use [pyproject.toml] for project and dependency management

    :white_check_mark: Use [setuptools] & [setuptools-scm] for packaging
    
    :white_check_mark: Use [tox] or [nox] for environment automation in testing

    :white_check_mark: Use [pytest] for testing framework
    
    :x: Avoid deeply nested namespaces (e.g. Java-style)


# Structure
```
repo
├── app
├── tests
│   └── unit_tests
│   └── integration_tests
├── .flake8
├── .gitignore
├── .pre-commit-config.yaml
├── .python-version
├── mypy.ini
├── pytest.ini
├── pyproject.toml
├── README.md
└── Makefile

```

[tox]: https://tox.readthedocs.io/
[nox]: https://nox.readthedocs.io/
[pytest]: https://docs.pytest.org/en/latest/
[setuptools]: https://setuptools.readthedocs.io/en/latest/
[setuptools-scm]: https://github.com/pypa/setuptools_scm/
[poetry]: https://python-poetry.org/
[pyproject.toml]: https://www.python.org/dev/peps/pep-0518/
