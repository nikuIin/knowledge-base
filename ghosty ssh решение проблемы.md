
[[Linux UNIX]]
Если вы подключаетесь к другому серверу по `ssh` и на нем не установлен `ghosty`, то вот решение: https://ghostty.org/docs/help/terminfo#ssh

Если кратко, то прокиньте на сервер `gosty`:

```shell
infocmp -x ghostty | ssh YOUR-SERVER -- tic -x -
```
