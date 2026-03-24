# Laravel Nginx UDS Lab

Тестовый проект для проверки сборки и запуска Laravel на базе boilerplate [`skufphp/laravel-starter-nginx-uds`](https://github.com/skufphp/laravel-starter-nginx-uds).

Репозиторий используется как лабораторный стенд для Laravel-приложения, собранного на связке:

- PHP-FPM + Nginx через Unix Socket
- PostgreSQL
- Redis
- Node.js / Vite
- Docker Compose для локальной разработки и production-like запуска

Отдельно проект подготовлен так, чтобы его было удобно использовать для деплоя через Dokploy: production-конфигурация уже вынесена в отдельные compose-файлы и ориентирована на контейнерный запуск.

## Что здесь проверяется

- Сборка Laravel-приложения из boilerplate-конфигурации
- Работа Nginx и PHP-FPM через Unix Socket вместо TCP
- Dev-режим с примонтированным кодом, Vite и Xdebug
- Production-сборка с multi-stage Dockerfile
- Запуск очередей и scheduler в отдельных контейнерах
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
- `docker-compose.prod.local.yml` — локальный запуск production-профиля
- `docker-compose.prod.yml` — production-конфигурация для серверного деплоя
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

Если нужна ручная последовательность, используйте:

```bash
make build
make up
make install-deps
make artisan CMD="key:generate"
make migrate
```

## Основные команды

### Development

```bash
make up
make down
make restart
make logs
make logs-php
make logs-nginx
make logs-postgres
make logs-redis
make logs-node
make status
```

### Laravel / PHP / Node

```bash
make artisan CMD="migrate"
make composer-install
make composer-update
make npm-install
make npm-dev
make npm-build
make test-php
make test-coverage
```

### Shell-доступ

```bash
make shell-php
make shell-nginx
make shell-node
make shell-postgres
make shell-redis
make shell-queue
make shell-scheduler
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
- Laravel запускается без dev-зависимостей
- frontend-ассеты собираются на этапе image build
- миграции выполняются автоматически при старте PHP-контейнера
- queue worker и scheduler вынесены в отдельные сервисы

## Dokploy

Проект подходит для деплоя через Dokploy как Docker Compose приложение.

Практически это означает:

- в качестве основной production-конфигурации можно использовать `docker-compose.prod.yml`
- переменные окружения следует задавать через `.env.production` или интерфейс Dokploy
- Nginx, PHP, PostgreSQL, Redis, queue worker и scheduler уже разделены по сервисам
- production-образы собираются из этого репозитория без необходимости отдельного Dockerfile для Dokploy

Перед деплоем через Dokploy проверьте:

- заполнены `APP_KEY`, `APP_URL` и production-переменные Laravel
- корректно настроены `DB_*` и `REDIS_*`
- выделены persistent volumes для PostgreSQL и Redis
- внешний роутинг Dokploy направлен на сервис `laravel-nginx-uds`

## Стек

- Laravel 13
- PHP 8.5 FPM Alpine
- Nginx
- PostgreSQL 18.2 Alpine
- Redis 8.6 Alpine
- Node.js 24 Alpine

## Примечания

- Dev-окружение использует bind mount проекта и удобно для локальной разработки.
- Production-конфигурация ориентирована на иммутабельную сборку контейнеров.
- Связка Nginx и PHP-FPM через Unix Socket повторяет архитектуру upstream boilerplate.

## Источники

- upstream README: <https://github.com/skufphp/laravel-starter-nginx-uds>
