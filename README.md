# ДЗ 3 — Docker

## 4 балла

Запуск одного контейнера:

```bash
docker build -t hw3-app .
docker run -d --name hw3-test -p 8080:8000 -e GREETING="Привет, Docker!" hw3-app
```

Проверка и логи:

```text
$ curl -fsS http://localhost:8080/
{"greeting": "Привет, Docker!", "count": 0, "hostname": "62a0203600a3", "storage": "sqlite"}

$ curl -fsS -X POST http://localhost:8080/increment
{"greeting": "Привет, Docker!", "count": 1, "hostname": "62a0203600a3", "storage": "sqlite"}

$ docker logs hw3-test
2026-11-12 12:30:34,673 listening port=8000 storage=sqlite
2026-11-12 12:33:28,376 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-12 12:34:48,151 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
```

## 6 и 8 баллов

Настройки порта и приветствия лежат в `.env.example` (`APP_PORT`, `GREETING`). Для 8 баллов используется Redis с сохранением данных на диск в `redis_data`.

Запуск:

```bash
docker compose up -d --build
```

### Проверки

1. Перезапуск без потери счётчика:

```text
$ curl -fsS -X POST http://localhost:8080/increment
{"greeting": "Hello, Docker!", "count": 1, "hostname": "a316f5eef9e4", "storage": "redis"}
$ curl -fsS -X POST http://localhost:8080/increment
{"greeting": "Hello, Docker!", "count": 2, "hostname": "a316f5eef9e4", "storage": "redis"}

$ docker compose down
$ docker compose up -d

$ curl -fsS http://localhost:8080/
{"greeting": "Hello, Docker!", "count": 2, "hostname": "208ffe8299db", "storage": "redis"}
```

2. Остановка Redis:

```text
$ docker compose stop redis

$ curl -i http://localhost:8080/healthz
HTTP/1.0 200 OK
{"status": "alive"}

$ curl -i http://localhost:8080/readyz
HTTP/1.0 503 Service Unavailable
{"error": "storage_unavailable"}

$ curl -i -X POST http://localhost:8080/increment
HTTP/1.0 503 Service Unavailable
{"error": "storage_unavailable"}
```

3. Запуск Redis обратно:

```text
$ docker compose start redis

$ curl -fsS http://localhost:8080/
{"greeting": "Hello, Docker!", "count": 2, "hostname": "208ffe8299db", "storage": "redis"}
$ curl -fsS -X POST http://localhost:8080/increment
{"greeting": "Hello, Docker!", "count": 3, "hostname": "208ffe8299db", "storage": "redis"}
```
