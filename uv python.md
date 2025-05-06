[[python]]

Install
```shell
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Download **python** version:

1.  See python versions list

```shell
uv python list
```

2. Install need python versionui

```shell
uv python install python-3.13.2
# use uninstall to delete
```

3. Pin this python version to the project

```shell
uv python pin 3.13.2 # .python-version file
```

_* if we want run our project in the other python version, we can change `.python_version` and set in this version that we needs_

### Init project (create `pyproject.toml`, `git`, `gitignore`, `readme`)

```shell
uv init
```

### Run python file

```shell
uv run main.py
```

### Add new dependencies 

```shell
uv add requests fastapi 

uv add --dev pytest # if we need dev-dependencies
```

### See install dependencies

```shell
uv pip list # in list (e.g. for requirements)

uv tree # beatiful tree-dependencies with versions
```

### Config pyrightconfig.json
We set this to LSP server.

```json
# pyrightconfig.json
{
	"venv": ".venv",
	"venvPath": "."
}
```


### Install old dependencies from requirements.txt

```shell
uv pip install -r requirements.txt
```
