!!! Summary

    :white_check_mark: Assume `make` is executed inside the virtual environment. 
    
    :white_check_mark: Wrap all virtual environment class inside `make`.
 
    :x: Avoid using calls that leaks the environment to system/global. 

# Tips

## The basics

Using `Makefile`, everything is based on the dependencies and timestamps.  If a dependnecy's timestamp is more recent than the target, then the rule is executed.

Take a look of this basic example.

```makefile
.venv/bin/python:
        python3 -m venv .venv

.venv/.install.stamp: .venv/bin/python pyproject.toml
        .venv/bin/python -m poetry install
        touch .venv/.install.stamp

test: .venv/.install.stamp
        .venv/bin/python -m pytest tests/
```

If the `pyproject.toml` file is changed when running `make test`, it will test the rule `.venv/.install.stamp` and detects there is a new timestamp, which cause this rule to be executed.  The dependency DAGs are traversed recursively. 


## Use variables

```makefile
VENV := .venv
INSTALL_STAMP := $(VENV)/.install.stamp
PYTHON := $(VENV)/bin/python

$(PYTHON):
        python3 -m venv $(VENV)

$(INSTALL_STAMP): $(PYTHON) pyproject.toml
        $(PYTHON) -m poetry install
        touch $(INSTALL_STAMP)

test: $(INSTALL_STAMP)
        $(PYTHON) -m pytest ./tests/
```

### Environment variables with defaults

Instead of hardcoding the name of your virtualenv folder, you can read it from the current shell environment and use a default value:

```makefile
VENV := $(shell echo $${VIRTUAL_ENV-.venv})
```

In Shell scripts, `${VAR_NAME-val}` will first try to get `$VAR_NAME` from the environment, if it can it will use the value that is set by the enviornment, otherwise, it will take the default `val`. 

You can pass this variable from the commandline as well

```bash
LOG_FORMAT=json make test

# or use export to have the variable in env:

export LOG_FORMAT=json && make test
```

### Check if a command is avaliable 

It is nice to check if certain command available before blindly execute them in run-time to discover it fails. 

```makefile
PY3 := $(shell command -v python3 2> /dev/null)

$(PYTHON):
        @if [ -z $(PY3) ]; then echo "python3 could not be found. See https://docs.python.org/3/"; exit 2; fi
        python3 -m venv $(VENV)
```

### List avaiable targets

when running `make`, by default it tries to execute `all` target.  This might not be intended.  We can customize this by doing

```makefile
.DEFAULT_GOAL := help

help:
        @echo "Please use 'make <target>' where <target> is one of"
        @echo ""
        @echo "  install     install packages and prepare environment"
        @echo "  format      reformat code"
        @echo "  lint        run the code linters"
        @echo "  test        run all the tests"
        @echo "  clean       remove *.pyc files and __pycache__ directory"
        @echo ""
        @echo "Check the Makefile to know exactly what each target is doing."
```

with `.DEFAULT_GOAL` it will be executing `help` instead of the `all`.


with a little shell script magic you can don't have to maintain the help message manually.

```makefile
.DEFAULT_GOAL := help

help:  ## Display this help
        @awk 'BEGIN {FS = ":.*##"; printf "\nUsage:\n  make \033[36m\033[0m\n\nTargets:\n"} /^[a-zA-Z_-]+:.*?##/ { printf "\033[36m%-10s\033[0m %s\n", $$1, $$2 }' $(MAKEFILE_LIST)

deps:  ## Check dependencies
        ...

clean: ## Cleanup the project folders
        ...

build: clean deps ## Build the project
        ...
```

## PHONY

`Make` by default assumes the target of a rule is a file.  If you have rules (tasks) that does not produce files on disk. (eg. `make clean` or `make test`), then you can mark them as `.PHONY`.  When a target is maked as `PHONY`, the targets are assumes to be *never up-to-date* and will always run when invoked.  

```makefile
.PHONY: clean
clean:
        rm -rf $(VENV)

.PHONY: test
test: ...
```



## Example

```makefile
NAME := ${PROJECT_NAME-myproject}
INSTALL_STAMP := .install.stamp
POETRY := $(shell command -v poetry 2> /dev/null)

.DEFAULT_GOAL := help

.PHONY: help
help:
        @awk 'BEGIN {FS = ":.*##"; printf "\nUsage:\n  make \033[36m\033[0m\n\nTargets:\n"} /^[a-zA-Z_-]+:.*?##/ { printf "\033[36m%-10s\033[0m %s\n", $$1, $$2 }' $(MAKEFILE_LIST)

install: $(INSTALL_STAMP)
$(INSTALL_STAMP): pyproject.toml poetry.lock
        @if [ -z $(POETRY) ]; then echo "Poetry could not be found. See https://python-poetry.org/docs/"; exit 2; fi
        $(POETRY) install
        touch $(INSTALL_STAMP)

.PHONY: clean
clean:
        find . -type d -name "__pycache__" | xargs rm -rf {};
        rm -rf $(INSTALL_STAMP) .coverage .mypy_cache

.PHONY: lint
lint: $(INSTALL_STAMP)
        $(POETRY) run isort --profile=black --lines-after-imports=2 --check-only ./tests/ $(NAME)
        $(POETRY) run black --check ./tests/ $(NAME) --diff
        $(POETRY) run flake8 --ignore=W503,E501 ./tests/ $(NAME)
        $(POETRY) run mypy ./tests/ $(NAME) --ignore-missing-imports
        $(POETRY) run bandit -r $(NAME) -s B608

.PHONY: format
format: $(INSTALL_STAMP)
        $(POETRY) run isort --profile=black --lines-after-imports=2 ./tests/ $(NAME)
        $(POETRY) run black ./tests/ $(NAME)

.PHONY: test
test: $(INSTALL_STAMP)
        $(POETRY) run pytest ./tests/ --cov-report term-missing --cov-fail-under 100 --cov $(NAME)

```
