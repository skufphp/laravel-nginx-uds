# Laravel Nginx UDS Lab

Тестовый проект для проверки сборки и запуска Laravel на базе boilerplate [`skufphp/laravel-starter-nginx-uds`](https://github.com/skufphp/laravel-starter-nginx-uds).

Репозиторий используется как лабораторный стенд для Laravel-приложения, собранного на связке:

- PHP-FPM + Nginx через Unix Socket
- PostgreSQL
- Redis
- Node.js / Vite
- Docker Compose для локальной разработки, локального production-like запуска и server-side production compose

Отдельно проект подготовлен так, чтобы его было удобно использовать для деплоя через Dokploy: production-конфигурация уже вынесена в отдельные compose-файлы и ориентирована на контейнерный запуск.

## Что здесь проверяется

- Сборка Laravel-приложения из boilerplate-конфигурации
- Работа Nginx и PHP-FPM через Unix Socket вместо TCP
- Dev-режим с примонтированным кодом, Vite и Xdebug
- Production-сборка с multi-stage Dockerfile
- Запуск очередей и scheduler в отдельных контейнерах
- Отдельные production compose-файлы для локальной проверки и серверного деплоя
- Готовность проекта к деплою через Docker Compose / Dokploy

## Основа проекта

Этот репозиторий основан на boilerplate-проекте:

- upstream: [`skufphp/laravel-starter-nginx-uds`](https://github.com/skufphp/laravel-starter-nginx-uds)

Из него взята общая архитектура окружения:

- Nginx проксирует запросы в PHP-FPM через Unix Socket
- PostgreSQL используется как основная БД
- Redis используется для кеша, сессий и очередей
- Node-контейнер отвечает за frontend-сборку и Vite HMR
- Production-сборка выполняется через multi-stage Dockerfile

## Структура

- `docker/` — Dockerfile и конфигурация PHP / Nginx
- `docker-compose.yml` — окружение для разработки
- `docker-compose.prod.local.yml` — локальный запуск production-профиля с публикацией `NGINX_PORT`
- `docker-compose.prod.yml` — production-конфигурация для серверного деплоя без проброса локальных портов
- `Makefile` — основные команды для разработки и обслуживания
- `.env.example` — шаблон переменных для dev
- `.env.production.example` — шаблон переменных для production

## Быстрый старт для разработки

1. Скопируйте переменные окружения:

```bash
cp .env.example .env
```

2. Заполните `.env` реальными значениями.

Минимально проверьте:

- `APP_*`
- `DB_*`
- `REDIS_*`
- `NGINX_PORT`
- `DB_FORWARD_PORT`
- `REDIS_FORWARD_PORT`
- `PGADMIN_*`
- `XDEBUG_*` при необходимости

3. Запустите инициализацию проекта:

```bash
make setup
```

Команда:

- соберет dev-образы
- поднимет контейнеры
- дождется готовности PostgreSQL и Redis
- установит Composer и NPM зависимости
- сгенерирует `APP_KEY`
- выполнит миграции
- выставит права на `storage/` и `bootstrap/cache`
- удалит `public/.htaccess` (не используется с Nginx)

После запуска будут доступны:

- Laravel: `http://localhost:8050` по умолчанию, порт берется из `NGINX_PORT`
- Vite dev server: `http://localhost:5173`
- pgAdmin: `http://localhost:8080` по умолчанию, порт берется из `PGADMIN_PORT`

Если нужна ручная последовательность, используйте:

```bash
make build
make up
make install-deps
make artisan CMD="key:generate"
make migrate
make cleanup-nginx
```

## Основные команды

### Development

```bash
make up
make down
make restart
make build
make rebuild
make logs
make logs-php
make logs-nginx
make logs-postgres
make logs-redis
make logs-node
make logs-queue
make logs-scheduler
make logs-pgadmin
make status
make validate
make info
```

### Laravel / PHP / Node

```bash
make artisan CMD="migrate"
make composer CMD="install"
make migrate
make rollback
make fresh
make tinker
make composer-install
make composer-update
make composer-require PACKAGE=vendor/package
make npm-install
make npm-dev
make npm-build
make test-php
make test-coverage
```

### Утилиты и очистка

```bash
make permissions
make cleanup-nginx
make clean
make clean-all
make dev-reset
```

### Shell и CLI-доступ

```bash
make shell-php
make shell-nginx
make shell-node
make shell-postgres
make shell-redis
make shell-queue
make shell-scheduler
```

`shell-postgres` запускает `psql`, а `shell-redis` выполняет проверку через `redis-cli`.

### Production / Production-like

```bash
make up-prod
make down-prod
make rebuild-prod
make logs-prod
make logs-php-prod
make logs-nginx-prod
make logs-postgres-prod
make logs-redis-prod
make logs-queue-prod
make logs-scheduler-prod
make shell-php-prod
make shell-nginx-prod
make shell-postgres-prod
make shell-redis-prod
make shell-queue-prod
make shell-scheduler-prod
make clean-prod
make clean-all-prod
make prod-reset
```

## Production-like локальный запуск

Для локальной проверки production-сценария:

1. Скопируйте шаблон:

```bash
cp .env.production.example .env.production
```

2. Заполните `.env.production`.

3. Запустите production-профиль:

```bash
make up-prod
```

Этот target использует:

- `.env.production`
- `docker-compose.prod.local.yml`
- production stage из `docker/php.Dockerfile` и `docker/nginx.Dockerfile`

Для остановки:

```bash
make down-prod
```

Для просмотра логов:

```bash
make logs-prod
```

В production-профиле:

- используется production stage из `docker/php.Dockerfile`
- используется production stage из `docker/nginx.Dockerfile`
- Laravel собирается без dev-зависимостей
- frontend-ассеты собираются на этапе image build
- миграции выполняются автоматически при старте PHP-контейнера
- перед стартом PHP выполняется `php artisan optimize:clear`
- queue worker и scheduler вынесены в отдельные сервисы
- локально публикуется только Nginx-порт, заданный через `NGINX_PORT`

## Dokploy

Проект подходит для деплоя через Dokploy как Docker Compose приложение.

Практически это означает:

- в качестве основной production-конфигурации для сервера следует использовать `docker-compose.prod.yml`
- `docker-compose.prod.local.yml` нужен именно для локальной проверки production-профиля
- переменные окружения следует задавать через `.env.production` или интерфейс Dokploy
- Nginx, PHP, PostgreSQL, Redis, queue worker и scheduler уже разделены по сервисам
- production-образы собираются из этого репозитория без необходимости отдельного Dockerfile для Dokploy

Перед деплоем через Dokploy проверьте:

- заполнены `APP_KEY`, `APP_URL` и production-переменные Laravel
- корректно настроены `DB_*` и `REDIS_*`
- выделены persistent volumes для PostgreSQL и Redis
- внешний роутинг Dokploy направлен на сервис `laravel-nginx-uds`
- если миграции не должны выполняться автоматически при каждом старте контейнера, скорректируйте `command` у `laravel-php-nginx-uds` в production compose

## Стек

- Laravel 13
- PHP 8.5 FPM Alpine
- Nginx 1.29 Alpine
- PostgreSQL 18.2 Alpine
- Redis 8.6 Alpine
- Node.js 24 Alpine

## Примечания

- Dev-окружение использует bind mount проекта и удобно для локальной разработки.
- В dev-окружении `laravel-node-nginx-uds` автоматически поднимает Vite на `0.0.0.0:5173`.
- Dev-профиль дополнительно включает `pgAdmin`, production-профили его не запускают.
- Production-конфигурация ориентирована на иммутабельную сборку контейнеров.
- Связка Nginx и PHP-FPM через Unix Socket повторяет архитектуру upstream boilerplate.

## Источники

- upstream README: <https://github.com/skufphp/laravel-starter-nginx-uds>
