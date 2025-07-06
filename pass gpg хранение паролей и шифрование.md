[[Linux UNIX]]
[[Удобные утилиты Linux Unix Mac. Удобные ништяки, команды. Тулзы linux]]

1. Создаем ключи
```shell
gpg --full-gen-key
```
2. Идем в `.gnupg` и создаем такой файл конфига `gpg.conf`
```gpg.conf
keyid-format 0xlong
throw-keyids
no-emit-version
no-comments
```
3. Получаем ключи (можно скипнуть)
```shell
gpg -k — публичные
gpg -K — приватные
```
Если хотим удалить:
```bash
gpg --delete-secret-key 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
```

4. Шифруем данные. Создаст зашифрованный файл `file.txt` -> `file.txt.asc`
```shell
gpg -e (encrypt, расшифровываем) -a (ascii, а не бинарник) -r vexyzy@yandex.ru file.txt
```
5. Расшифровываем
```shell
gpg -d (decrypt) -o file.txt (выходной файл) file.txt.asc
```
6. Экспортируем ключи и заносим на флешку
```shell
gpg --export -a vexyzy@yandex.ru > public.gpg
gpg --export-secret-key -a vexyzy@yandex.ru > private.gpg
```
7. Импортируем ключи c флешки
```shell
gpg --import public.gpg
gpg --import secret.gpg
```
8. Инициализируем `pass`
```shell
pass init vexyzy@yandex.ru
```
9. Добавляем свой пароль в `Services/eva_vpn`
```shell
pass insert Services/eva_vpn
```
10. Добавляем несколько строк (храним и логин и пароль):
```shell
pass insert Servicis/some -m
```
11. Получаем пароль
```shell
pass Services/some
```