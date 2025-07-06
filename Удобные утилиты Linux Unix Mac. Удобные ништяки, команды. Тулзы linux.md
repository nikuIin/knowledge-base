l
[[Linux UNIX]]

Такс:

1. Релоднуть терминал без выхода (обновить `.zshrc` например)
```bash
omz reload
```

2. Быстрый поиск с превью (у меня алиас `fbf`)
```shell
fzf --preview="bat {} --color=always"
```

3. ну просто шикарный аналог `ls cd`: `yazi` (у меня алиас `yz`)
```shell
yazi
```

4. pass gpg
[[pass gpg хранение паролей и шифрование]]

5. Копируем в буфер командой
```shell
pbcopy

# Например:
pass | pbcopy
```

6. [[генерируем ssh ключи в терминале]]
7. Открываем в `Finder`
```shell
open .
```

8. `tabview` – удобный просмотр `csv`, `json` и других структуризированных файлов:
```bash
tw path-to-file
```

9. Утилита для быстрого чтения — `speedread`
```bash
cat path-to-read-file | speedread -w 300 # ~300 слов в минуту
```

* *git status* = *st* = *git st*
* *git lg*  = *git log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --branches*

А это с комментария курса
---
#### Аналоги *nix утилит на Rust

ls — [eza](https://github.com/eza-community/eza)  
cd — [zoxide](https://github.com/ajeetdsouza/zoxide) – более умная команда cd  
cat — [bat](https://github.com/sharkdp/bat)  
find — [fd](https://github.com/sharkdp/fd) – простая, быстрая и удобная альтернатива “find”  
grep — [ripgrep](https://github.com/BurntSushi/ripgrep) – инструмент поиска, рекурсивно ищет в каталогах шаблон регулярных выражений, учитывая `.gitignore`  
набор утилит [fuc](https://github.com/SUPERCILEX/fuc):

- cp — [cpz](https://github.com/SUPERCILEX/fuc/tree/master/cpz) – инструмент копирования файлов и каталогов, удобная альтернатива
    
- rm — [rmz](https://github.com/SUPERCILEX/fuc/tree/master/rmz) – инструмент для удаления файлов и каталогов, удобная альтернатива
    

tree — [broot](https://github.com/Canop/broot) – новый способ просмотра деревьев каталогов и навигации по ним  
top — [zenith](https://github.com/bvaisvil/zenith) – диспетчер задач, с графиками и информацией об использовании CPU, GPU, сети и диска.