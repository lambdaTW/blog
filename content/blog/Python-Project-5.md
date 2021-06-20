+++
title = "Python Project (5) Make Automatically"
author = "lambda@lambda.tw"
categories = ["python", "project"]
tags = ["python", "project"]
date = "2021-04-27"
description = "Automatic everythings with Makefile"
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
type = "post"
+++
# Makefile
Makefile 是個很古老的東西，可以把很多東西自動化，以前用來 compile 需要編譯的語言，但是做為自動化的語意，是十分優雅的，這邊我們把之前有用到的套件，利用 `Makefile` 將他自動化

## Usage
```shell-script
make <target>
```

## Configuration
```Makefile
PKG_FILES := $(shell ls requirements/*.txt)

build:                          ## Build this project as pip wheel
	rm -rf dist/*
	python setup.py bdist_wheel
.PHONY: build

dev-install: build              ## Install current code into venv
	pip install -U dist/*.whl

pip:                            ## Recompile and install all pip packages
	pip-compile requirements/base.in
	pip-compile --generate-hashes requirements/development.in
	pip-compile --generate-hashes requirements/deployment.in
	pip-sync $(PKG_FILES)
.PHONY: pip

test: dev-install               ## Run test only
	pytest tests
.PHONY: test

typecheck:                      ## Run typechecking
	python -m mypy --show-error-codes --pretty src
.PHONY: typecheck

lint:                           ## Run linting
	python -m black --check .
	python -m isort -c .
	python -m flake8 .
	python -m pydocstyle .
.PHONY: lint

ci: typecheck lint test         ## Run all checks (test, lint, typecheck)
.PHONY: ci

tox:
	tox -p all
.PHONY: tox

lint-fix:                       ## Run autoformatters
	python -m black .
	python -m isort .
.PHONY: lint-fix

push:                           ## Push code with tags
	git push && git push --tags

.DEFAULT_GOAL := help
help: Makefile                  ## Show Makefile help
	@echo "Below shows Makefile targets"
	@grep -E '(^[a-zA-Z_-]+:.*?##.*$$)|(^##)' Makefile | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[32m%-30s\033[0m %s\n", $$1, $$2}' | sed -e 's/\[32m##/[33m/'
```
