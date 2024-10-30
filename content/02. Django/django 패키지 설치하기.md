---
title: Django 패키지 설치하기
publish: false
tags:
---
### requirements.in
```requirements.in
-r base.in
pytest
black
flake8
```

```sh
pip-compile requirements.in
```
### poetry
```pyproject.toml
[tool.poetry]  
name = "project_name"  
version = "0.1.0"  
description = "description"  
authors = []  
readme = "../README.md"  
  
[tool.poetry.dependencies]  
python = "^3.11"  
  
# django  
django = "^4.2"  
psycopg2-binary = "^2.9.6"  
django-environ = "^0.9.0"  
djangorestframework = "^3.14.0"  
django-cors-headers = "^3.13.0"  
factory-boy = "^3.2.1"  
Faker = "^18.7.0"  
  
django-debug-toolbar = "^4.2.0"  
pytest-django = "^4.7.0"  
  
# deploy  
gunicorn = "^22.0.0"  
  
ipdb = "^0.13.13"  
  
[tool.poetry.group.dev.dependencies]  
black = "^24.3.0"  
isort = "^5.13.2"  
ipdb = "^0.13.13"  
pre-commit = "^3.8.0"  
  
[build-system]  
requires = ["poetry-core"]  
build-backend = "poetry.core.masonry.api"

[tool.black]
line-length = 119
target-version = [
  'py312',
]

# ==== isort ====

[tool.isort]
profile = "black"
line_length = 119
known_first_party = [
  "tests",
  "scripts",
  "hooks",
]

# ==== pytest ====

[tool.pytest.ini_options]
addopts = "-v --tb=short"
norecursedirs = [
  ".tox",
  ".git",
  "*/migrations/*",
  "*/static/*",
  "docs",
  "venv",
  "*/my_project/*",
]
```
[[poetry]]
## package settings
### pre-commit
```.pre-commit-config.yaml
exclude: "my_project"
default_stages: [pre-commit]

default_language_version:
  python: python3.12

repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-json
      - id: check-toml
      - id: check-xml
      - id: check-yaml
      - id: debug-statements
      - id: check-builtin-literals
      - id: check-case-conflict
      - id: detect-private-key

  - repo: https://github.com/pre-commit/mirrors-prettier
    rev: "v4.0.0-alpha.8"
    hooks:
      - id: prettier
        args: ["--tab-width", "2"]

  - repo: https://github.com/psf/black
    rev: 24.10.0
    hooks:
      - id: black

  - repo: https://github.com/PyCQA/isort
    rev: 5.13.2
    hooks:
      - id: isort

  - repo: https://github.com/PyCQA/flake8
    rev: 7.1.1
    hooks:
      - id: flake8

  - repo: https://github.com/tox-dev/pyproject-fmt
    rev: "2.4.3"
    hooks:
      - id: pyproject-fmt

ci:
  autoupdate_schedule: weekly
  skip: []
  submodules: false
```