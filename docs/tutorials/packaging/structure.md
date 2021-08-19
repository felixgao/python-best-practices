!!! Summary

    :white_check_mark: Use [pyproject.toml] for project and dependency management

    :white_check_mark: Use [poetry] for packaging and releasing
    
    :white_check_mark: Use [tox] or [nox] for environment automation in testing

    :white_check_mark: Use [pytest] for testing framework
    
    :x: Avoid deeply nested namespaces (e.g. Java-style)


# Structure

for simple projects.

```
.
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

If your project is rather large, you may opt for the monorepo pattern

```
.
├── README.md
├── libs
│   ├── common-lib
│   ├── lib-one
│   └── lib-one
├── poetry.lock
├── projects
│   ├── __init__.py
│   ├── project-one
│   └── project-two
└── pyproject.toml
```

under `/projects`

Project code (Python modules) go here.
Each project has its own dependencies. 
Each project is considered to have its own releasable artifact.

under `/libs`

Each lib has its own dependencies.
Each lib can optionally depends on the `common-lib` if there one.

project under `/projects` will use path import of the lib under `/libs`.



[tox]: https://tox.readthedocs.io/
[nox]: https://nox.readthedocs.io/
[pytest]: https://docs.pytest.org/en/latest/
[setuptools]: https://setuptools.readthedocs.io/en/latest/
[setuptools-scm]: https://github.com/pypa/setuptools_scm/
[poetry]: https://python-poetry.org/
[pyproject.toml]: https://www.python.org/dev/peps/pep-0518/
