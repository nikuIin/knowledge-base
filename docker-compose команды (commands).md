[[Docker.canvas|Docker]]
[[docker-compose.yaml]]

- Билд проекта и запуск проекта (`-d` — запуск в `detach` режиме):

```bash
docker compose up -d
```

- Только билд (ребилд) проекта — создаем `образы` (`image`):
  `--no-cache` — флаг, указывающий на неиспользование кэша

```bash
docker compose build
```

- Билд (ребилд) отдельного образа

```bash
docker compose build path-to-image-directory
```

- Остановка `docker-compose`

```bash
docker compose stop
```

- Старт `docker-compose`

```bash
docker compose start
```

- Остановка и удаление контейнеров

```bash
docker compose down
```

- Остановка и удаления всех зависимостей (`images`) и флагом `-v` — удаление `volumes`

```bash
docker compose down --rmi all -v
```