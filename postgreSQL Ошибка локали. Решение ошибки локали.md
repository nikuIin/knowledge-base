[[postgres/postgreSQL|postgreSQL]]
[[PostgreSQL DB.canvas|PostgreSQL DB]]

Решение ошибки **`ERROR: invalid locale name: "ru_RU.UTF-8"`**

1. Генерируем новую локаль (если не работает, то нужно раскоментить ее в `/etc/locale.gen`):

```bash
locale-gen ru_RU.UTF-8
```

2. Конфигурируем (выдастся список локали. На 09.07.2025 русская локаль №241, иногда стоит и указать английскую локаль сразу (№97)):
```bash
dpkg-reconfigure locales
```

3. Смотрим все кластеры
```bash
pg_lsclusters
# Ver Cluster Port Status Owner    Data directory               Log file
# 9.5 main    5432 online postgres /var/lib/postgresql/17/main # /var/log/postgresql/postgresql-17-main.log
```

4. Удаляем кластер:
```bash
pg_dropcluster --stop 17 main
# Redirecting stop request to systemctl
```

5. Создаём новый
```bash
pg_createcluster --locale ru_RU.utf8 --start 17 main
# Creating new cluster # 9.5/main ...   config /etc/postgresql/9.5/main   data   # #/var/lib/postgresql/9.5/main   locale ru_RU.utf8   socket  #/var/run/postgresql   port   5432 Redirecting start request to systemctl
```
