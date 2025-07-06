[[Linux UNIX]]

Создаем скрипт `ytr` для `CLI` перевода яндекса

1. Работаем с репо: https://github.com/nikuIin/ytr

Создадим поверх скрипта `ytr.py` мета-скрипт-блок для `uv`:

```python
# /// script
# requires-python = ">=3.13"
# dependencies = [
#   "httpx",
#   "click",
#   "rich",
#   "typer"
# ]
# ///


import uuid
from typing import Any, TypedDict
import click
...
```

Теперь мы можем запускать `uv run ytr.py` и он подтянет все необходимые зависимости.

2. Создаем `bash` скрипт с разными (опираясь на скрипт) параметрами запуска:

```bash
#!/bin/bash

# Определяем путь к ytr.py относительно скрипта
SCRIPT_DIR="/Users/ivanaleksandrovci/cli-tools/ytr"
YTR_PATH="$SCRIPT_DIR/ytr.py"

# Проверяем наличие ytr.py
if [ ! -f "$YTR_PATH" ]; then
    echo "Error: ytr.py not found at $YTR_PATH"
    exit 1
fi

# Проверяем, передан ли флаг -w и аргумент
if [ "$1" = "-w" ] && [ -n "$2" ]; then
    uv run --quiet "$YTR_PATH" -w "$2"
else
    # Проверяем, что команда вызвана без аргументов или с некорректными
    if [ $# -eq 0 ] && [ "$1" != "-w" ]; then
        uv run "$YTR_PATH"
    else
        echo "Usage: ytr [-w word]"
        exit 1
    fi
fi
```

3. Переносим этот скрипт в `/opt/homebrew/bin` и даем права на `x`

```bash
chmod +x ytr.sh
mv ytr.sh /opt/homebrew/bin/ytr
```

4. Проверяем работоспособность скрипта:
```bash
ytr -w "Hello world!" 
# -> Привет мир!
```

5. Создаем `script` в `~/raycast-scripts`
```bash
#!/bin/bash

# Required parameters:
# @raycast.schemaVersion 1
# @raycast.title Translate
# @raycast.mode fullOutput

# Optional parameters:
# @raycast.icon 🤖
# @raycast.argument1 { "type": "text", "placeholder": "Enter text to translate", "optional": false }

# Documentation:
# @raycast.description CLI Yandex Translate tool
# @raycast.author van

# Проверяем, доступна ли команда ytr
if ! command -v ytr >/dev/null 2>&1; then
    echo "Error: ytr command not found. Please ensure it is installed in /usr/homebrew/bin."
    exit 1
fi

# Выполняем перевод
ytr -w "$1"
```