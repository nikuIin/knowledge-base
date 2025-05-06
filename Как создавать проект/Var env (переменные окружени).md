
```bash
true > setenv.sh # ../src
```

```shell
# <--gitignore-->

# all hide files and directories instead .gitignore
.*
.*/
!.gitignore

# virtual enviroments script
setenv.sh
```

```bash
# <--setenv.sh-->
export API_KEY=abc
export DB_PASSWORD=cool_password
export DB_NAME=tests
...
```

```python
from os import getenv
API = getenv("API_KEY")
DB_NAME = getenv("DB_NAME")
...
```
