
_pyproject.toml_

```toml
[project]
name = "test"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.13"
dependencies = []

[tool.ruff]
line-length = 100
src = ["src"]

# rooles: https://docs.astral.sh/ruff/rules/
select = [
	"E",    # pycodestyle errors
    "W",    # pycodestyle warnings
    "F",    # pyflakes
    "I",    # isort
    "C",    # flake8-comprehensions
    "B",    # flake8-bugbear
    "SIM",  # flake8-simplify
    "UP",   # pyupgrade
    "N",    # pep8-naming
	"UP", # pyupgrade
	"N", # pep8-naming
	"ERA", # found commented-out code
	"D401", # check docstring format
]

ignore = [
    "B008", # Ignore depends in function calls
    "E731", # Allow to use the lambda functions
](например, "E501" для длинных строк, если ruff все равно ругается)
exclude = [".venv", ".git", "__pycache__", "build", "dist"]

[tool.black]
# Black is disabled by omitting its configuration here.

[tool.pyright]
# Delete if you dont have mistakes
executionEnvironments = [{ root = "src" }]
typeCheckingMode = "standard"
venv = "env"
venvPath = "."
```

