[[Linux UNIX]]

Команда для работы с архивами типа `rar`

```shell
brew install rar

# или
sudo install -c -o $USER rar /bin
sudo install -c -o $USER unrar /bin
```

Создать архив каталога:
```shell
rar a archive.rar folder/
```

Распаковка архива в текущую директорию:

```shell
unrar e archive.rar
```
