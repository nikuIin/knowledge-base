Обновление репозиториев и поиск в них чего-то похожего на `postgres`:

```
sudo apt update
apt search postgres | less
```

Проверяем версию Debian:

```
cat /etc/os-release
```

Гугл по запросу «debian bookworm postgresql 16» приводит нас [сюда](https://www.postgresql.org/download/linux/debian/). Выполняем операции из документации по ссылке:

```
# Import the repository signing key:
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc

# Create the repository configuration file:
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Update the package lists:
sudo apt update

# Install the latest version of PostgreSQL:
# If you want a specific version, use 'postgresql-16' or similar instead of 'postgresql'
sudo apt -y install postgresql
```

Если система ругается на нехватку `lsb_release`, просто поставь его:

```
sudo apt install lsb-release
```

Проверяем в `htop`, что `postgres` работает.

Пробуем зайти в `psql`-клиент, с помощью которого мы можем отправлять команды к СУБД и смотреть ответы на эти команды:

```
psql -h localhost -U postgres
```

Сбрасываем пароль на Linux-пользователя `postgres` (устанавливая новый пароль), заходим в Linux под ним (под Linux-пользователем `postgres`) и запускаем `psql`:

```
sudo passwd postgres
su - postgres
psql
```

Проверим работающую версию сервера PostgreSQL, к которому мы сейчас подключились, выполнив в `psql`:

```
select version();
```

Отлично!

Создадим роль и базу данных (поговорим подробнее об этом позже, а сейчас надо просто создать), выполняем в `psql`:

```
CREATE ROLE rroom WITH LOGIN PASSWORD 'mycoolp@ssword';

CREATE DATABASE rroom_db
WITH
ENCODING='UTF8'
LC_COLLATE='ru_RU.UTF-8'
LC_CTYPE='ru_RU.UTF-8'
owner rroom;
```

Если создание базы данных не проходит, попробуйте указать шаблон создаваемой базы вот так:

```
CREATE DATABASE rroom_db
WITH
TEMPLATE=template0
ENCODING='UTF8'
LC_COLLATE='ru_RU.UTF-8'
LC_CTYPE='ru_RU.UTF-8'
owner rroom;
```

Здесь мы создали роль `rroom` с паролем `mycoolp@ssword` и создали базу данных `rroom_db`. Логин, пароль и базу данных стоит запомнить — те, что вы задаёте у себя; вы, конечно, можете назвать их как угодно и использовать свой пароль.

Чтобы посмотреть, где лежат данные наших баз данных (если интересно), выполняем в `psql`:

```
show data_directory;
```

Чтобы настроить подключение к PostgreSQL «снаружи» виртуальной машины, то есть из домашней операционной системы (в моём случае Mac OS), необходимо провести ряд настроек. Проверяем в `psql`, где лежат конфиги:

```
SHOW config_file;
SHOW hba_file;
```

Выходим из `psql`, нажав `CTRL`+`D`, выходим из Linux-сессии Linux-пользователя `postgres`, нажав `CTRL`+`D`, и от нашего обычного Linux-пользователя работаем дальше.

Правим первый конфиг (можете воспользоваться любым приятным для вас консольным текстовым редактором):

```
sudo vim /etc/postgresql/16/main/postgresql.conf
```

В файле необходимо установить параметр:

```
listen_addresses = '*'
```

В файле скорее всего будет закомментированная строка c `listen_addresses`, её надо раскомментировать, удалив первый символ строки `#`, и затем установить значение этого параметра, как показано выше.

Затем редактируем второй конфиг:

```
sudo vim /etc/postgresql/16/main/pg_hba.conf
```

В этот файл надо в конец добавить строку:

```
host all rroom 0.0.0.0/0 md5
```

 Здесь мы разрешаем коннект (подключение) с любого IP по логину и паролю для пользователя `rroom`. На продакшн как правило нам нужна другая настройка, но мы сейчас настраиваем локальную development-среду для собственного локального использования.

Рестартим сервер PostgreSQL (перезагружаем его):

```
sudo service postgresql restart
```

Смотрим айпишник сервера, чтобы подключиться к серверу «снаружи», из домашней ОС (в моём случае Mac OS):

```
ip a
```

В моём случае IP показался такой `192.168.64.11`, из домашней ОС (в моём случае Mac OS) устанавливаем `psql` и подключаемся к серверу PostgreSQL, запущенному в виртуальной машине с Linux под этим IP:

```
brew install libpq
psql -h 192.168.64.11 -U rroom -d rroom_db
```

Обратите внимание — `psql` с `brew` ставится именно так.

`psql` на Windows можно установить с [PgAdmin](https://www.pgadmin.org/download/pgadmin-4-windows/). Директорию с `psql` затем можно добавить в PATH на Windows. Жмём Пуск, вводим: "Изменение переменных среды текущего пользователя", в открывшемся интерфейсе в PATH добавляем путь до директории с `psql`, в моём случае это `C:\Program Files\pgAdmin 4\runtime\`.