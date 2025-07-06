Все о git
[[Git.canvas|Git]]

## Конфиг
`global config path`: `~/.gitconfig`
`loacl config path`: `/path-to-repository/.git/config`

Чтобы открыть конфигурацию:
```bash
git config --global -e
```

```toml
[filter "lfs"]
	smudge = git-lfs smudge -- %f
	process = git-lfs filter-process
	required = true
	clean = git-lfs clean -- %f
[user]
	name = van
	email = vexyzy@yandex.ru
[init]
    defaultBranch = main
[alias]
    lg = log --all --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --branches
```

### Выводим конфиг с пометками origin (к какому конфигу относится параметр)

```bash
git config --list --show-origin
```

Важно понимать, что **локальный** конфиг ==предпочтительнее== **глобального**.
Но `git` считывает сначала глобальный, а потом локальный. Локальный может переопределить настройки.
# Алиасы

**Важно**: алиас не должен совпадать с самой командой `git`.

Алиасы ставятся командой:

```bash
git config --global alias.loga "log --oneline --graph --all"
```
либо в конфигурационном файле в блоке `alias`

* `git lg` — красивый лог 
* `git lga` — красивый полный лог (с пометкок `-all`)
* `git st`;`st` — сокращенный `git status`
* `lg` — `lazygit`
* `git br` — `git branch -vv`
* `git bra` — `git branch --all -vv`
* `git c` — `git commit`