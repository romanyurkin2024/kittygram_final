# Kittygram

Социальная сеть для любителей котиков: можно добавить своего кота, загрузить его фото, указать год рождения, цвет и достижения, а также посмотреть котиков других пользователей.

Проект запущен в Docker-контейнерах и автоматически разворачивается на сервере через GitHub Actions при каждом пуше в ветку `main`.

Сайт: https://romankitty.duckdns.org

## Стек

Python 3.12 · Django 5 · Django REST Framework · Djoser · PostgreSQL 13 · Gunicorn · Nginx · React · Docker / Docker Compose · GitHub Actions

## Как устроено

```
браузер → nginx на сервере (HTTPS) → :9000 → gateway (nginx в контейнере)
                                              ├─ /api/, /admin/ → backend (Gunicorn + Django) → db (PostgreSQL)
                                              ├─ /media/        → том media (фото котиков)
                                              └─ /              → том static (React + статика админки)
```

## Локальный запуск в Docker

1. Клонировать репозиторий:
   ```bash
   git clone git@github.com:romanyurkin2024/kittygram_final.git
   cd kittygram_final
   ```
2. Создать `.env` в корне по образцу `.env.example`.
3. Запустить контейнеры:
   ```bash
   docker compose up --build
   ```
4. В соседнем терминале выполнить миграции и собрать статику:
   ```bash
   docker compose exec backend python manage.py migrate
   docker compose exec backend python manage.py collectstatic --noinput
   docker compose exec backend cp -r /app/collected_static/. /backend_static/static/
   ```
5. Открыть http://localhost:9000

Создать администратора: `docker compose exec backend python manage.py createsuperuser`

## Переменные окружения

| Переменная | Описание |
|---|---|
| `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` | имя базы, пользователь и пароль PostgreSQL |
| `DB_HOST`, `DB_PORT` | адрес и порт базы (`db`, `5432`) |
| `SECRET_KEY` | секретный ключ Django |
| `DEBUG` | режим отладки (`False` на сервере) |
| `ALLOWED_HOSTS` | разрешённые адреса через запятую |
| `CSRF_TRUSTED_ORIGINS` | адрес сайта с `https://` (для входа в админку) |

## CI/CD

При пуше в `main` GitHub Actions:
1. проверяет код бэкенда flake8 и запускает тесты бэкенда (с PostgreSQL) и фронтенда;
2. собирает образы `kittygram_backend`, `kittygram_frontend`, `kittygram_gateway` и отправляет их на Docker Hub;
3. подключается к серверу по SSH, скачивает новые образы и перезапускает контейнеры, выполняет миграции и собирает статику;
4. присылает сообщение в Telegram об успешном деплое.

Секреты репозитория: `DOCKER_USERNAME`, `DOCKER_PASSWORD`, `HOST`, `USER`, `SSH_KEY`, `SSH_PASSPHRASE`, `TELEGRAM_TO`, `TELEGRAM_TOKEN`.

## Автор

Роман — [GitHub](https://github.com/romanyurkin2024)
