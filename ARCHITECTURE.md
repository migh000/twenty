# Архитектура Twenty

Документ описывает проект Twenty: его цель, ключевые компоненты, технологический стек и способы локального запуска.

## Что за проект

**Twenty** — открытая CRM-система (Open-Source CRM), позиционируемая как альтернатива закрытым продуктам (Salesforce, HubSpot и т.п.). Лицензия — AGPL-3.0.

**Цель проекта:** дать командам и разработчикам конструктор CRM, который можно настраивать как код:

- Объекты, поля, представления, рабочие процессы и AI-агенты описываются как код через `twenty-sdk`.
- CRM версионируется и поставляется как часть остальной кодовой базы продукта.
- Систему можно использовать в облаке (`twenty.com`) или развернуть self-hosted через Docker Compose.

Ключевые продуктовые возможности:

- Кастомизируемые объекты и поля (контакты, компании, сделки и любые свои сущности).
- Представления (таблицы, канбан, формы) с гибкой конфигурацией.
- AI-агенты и чаты, интегрированные в CRM.
- Workflow / автоматизации.
- REST/GraphQL API + SDK для расширений.

## Технологический стек

| Слой | Технологии |
| --- | --- |
| Язык | TypeScript |
| Монорепозиторий | Nx workspace + Yarn 4 |
| Backend | NestJS, TypeORM, GraphQL (Yoga, code-first), BullMQ (фоновые задачи) |
| Frontend | React 18, Vite, Jotai (state), Linaria (стилизация), Lingui (i18n) |
| База данных | PostgreSQL (основная), Redis (кэш/очереди), ClickHouse (аналитика, опционально) |
| Маркетинговый сайт | Next.js (App Router) |
| Email-шаблоны | React Email |
| E2E-тесты | Playwright |
| Интеграции | Zapier |
| Среда исполнения | Node.js ^24.5.0 |

## Структура монорепозитория

Все пакеты лежат в `packages/`:

| Пакет | Назначение |
| --- | --- |
| `twenty-server` | Backend API на NestJS (GraphQL, REST, фоновый воркер) |
| `twenty-front` | Frontend SPA (React + Vite) — основное приложение CRM |
| `twenty-ui` | Библиотека общих UI-компонентов |
| `twenty-shared` | Общие типы и утилиты (используются и фронтом, и сервером) |
| `twenty-emails` | Email-шаблоны (React Email) |
| `twenty-website` | Маркетинговый сайт `twenty.com` (Next.js) |
| `twenty-docs` | Сайт документации |
| `twenty-sdk` / `twenty-client-sdk` | SDK для определения объектов, полей, представлений как кода |
| `twenty-cli` / `create-twenty-app` | CLI для скаффолдинга и публикации приложений |
| `twenty-zapier` | Интеграция с Zapier |
| `twenty-e2e-testing` | Playwright-тесты сценариев конца-в-конец |
| `twenty-docker` | Конфигурация Docker / Docker Compose для разработки и self-hosting |
| `twenty-utils` | Утилиты и скрипты разработки (включая `setup-dev-env.sh`) |
| `twenty-front-component-renderer` | Раннер для пользовательских React-компонентов |
| `twenty-codex-plugin`, `twenty-companion`, `twenty-claude-skills` | Плагины / интеграции AI-инструментов |
| `twenty-oxlint-rules`, `twenty-apps` | Лин-правила и шаблоны приложений |

## Основные компоненты и их связи

### Высокоуровневая схема

```
┌────────────────────────┐         ┌──────────────────────────┐
│  twenty-front (React)  │ ◀────▶ │   twenty-server (NestJS) │
│  - Jotai (state)       │ GraphQL │   - GraphQL (Yoga)       │
│  - Apollo Client       │  / REST │   - REST endpoints       │
│  - Linaria, Lingui     │         │   - TypeORM (Postgres)   │
└──────────┬─────────────┘         │   - BullMQ (Redis)       │
           │                       │   - Worker (фоновые джобы)│
           │ shared types          └─────────┬────────────────┘
           ▼                                  │
   ┌────────────────┐                         ▼
   │ twenty-shared  │           ┌──────────────────────────┐
   │ (types/utils)  │           │  PostgreSQL │ Redis │ CH │
   └────────────────┘           └──────────────────────────┘
```

### Backend (`twenty-server`)

- **NestJS-модули** организуют фичи (auth, метаданные, workspace-сущности, биллинг и т.д.).
- **TypeORM** управляет схемой PostgreSQL и миграциями.
- **GraphQL (code-first)** на базе GraphQL Yoga — основной API для фронта.
- **BullMQ + Redis** обслуживают очереди фоновых задач (отправка писем, синхронизация календарей/почты и т.д.). Запускаются как отдельный воркер: `twenty-server:worker`.
- **Многотенантность:** в PostgreSQL поддерживается несколько схем — `core`, `metadata` и схемы конкретных workspace'ов. Метаданные позволяют генерировать GraphQL-схему на лету под каждый workspace.
- **Instance / Workspace команды** (`@RegisteredInstanceCommand`, `@RegisteredWorkspaceCommand`) — механизм апгрейда: после изменения сущностей команда генерируется через `database:migrate:generate`, выполняется на инстансе либо по всем workspace'ам. Подробнее: `packages/twenty-server/docs/UPGRADE_COMMANDS.md`.

### Frontend (`twenty-front`)

- **React 18 + Vite** — основное приложение CRM.
- **Jotai** — глобальный state (атомы, селекторы, atom families для динамических коллекций).
- **Apollo Client** — кэш и работа с GraphQL.
- **Linaria** — zero-runtime CSS-in-JS.
- **Lingui** — интернационализация.
- Сгенерированные GraphQL-типы обновляются командой `npx nx run twenty-front:graphql:generate`.

### Совместные пакеты

- `twenty-shared` — общие типы и утилиты (`isDefined`, `isNonEmptyString`, `isNonEmptyArray` и т.п.) для фронта и сервера. Должен собираться первым.
- `twenty-ui` — UI-компоненты, переиспользуемые фронтом и Storybook.
- `twenty-sdk` / `twenty-client-sdk` — описывают объекты, поля и представления для расширения CRM как кода (см. `npx create-twenty-app`).

### Поток данных

1. Пользователь в `twenty-front` вызывает GraphQL-запрос.
2. `twenty-server` обрабатывает запрос: проходит auth, через TypeORM работает с PostgreSQL (workspace-схема + метаданные).
3. Долгие операции уходят в Redis-очередь BullMQ; их выполняет `twenty-server:worker`.
4. Изменения сущностей метаданных триггерят перегенерацию GraphQL-схемы для конкретного workspace.

## Локальный запуск

Минимальные требования:

- Node.js `^24.5.0`
- Yarn `>=4.0.2`
- Docker (опционально, если не используется локальный Postgres/Redis)

### Шаг 1. Установка зависимостей

```bash
yarn install
```

### Шаг 2. Подготовка окружения (БД, Redis, env, миграции)

Один скрипт делает всё: поднимает Postgres + Redis (автоматически выбирает локальные сервисы или Docker), создаёт базы, копирует `.env`, инициализирует схему. Идемпотентен.

```bash
bash packages/twenty-utils/setup-dev-env.sh
```

Полезные флаги:

- `--docker` — принудительно использовать Docker (`packages/twenty-docker/docker-compose.dev.yml`).
- `--down` — остановить сервисы.
- `--reset` — полностью пересоздать данные.

### Шаг 3. Запуск приложения

Поднять backend + frontend + воркер одной командой:

```bash
yarn start
```

Эквивалент по частям:

```bash
npx nx start twenty-server               # backend (NestJS)
npx nx start twenty-front                # frontend (Vite)
npx nx run twenty-server:worker          # фоновый воркер
```

После запуска фронт доступен по адресу из `.env` (обычно `http://localhost:3001`), backend — по `http://localhost:3000`.

### Шаг 4. Проверка работы

При тестировании UI достаточно нажать **Continue with Email** — учётные данные подставляются автоматически.

### Полезные команды разработки

```bash
# Тесты
npx nx test twenty-front
npx nx test twenty-server
npx nx run twenty-server:test:integration:with-db-reset

# Линт и типы
npx nx lint:diff-with-main twenty-front
npx nx typecheck twenty-server

# GraphQL-типы (после изменения схемы)
npx nx run twenty-front:graphql:generate

# Сборка (twenty-shared собирается первым)
npx nx build twenty-shared
npx nx build twenty-front
npx nx build twenty-server

# Миграции
npx nx database:reset twenty-server
npx nx run twenty-server:database:migrate:generate --name <name> --type <fast|slow>
```

### Self-hosting

Для развёртывания вне разработки есть Docker Compose-конфигурация в `packages/twenty-docker` и [официальное руководство по self-host](https://docs.twenty.com/developers/self-host/capabilities/docker-compose).

## Дополнительные ресурсы

- Основной README: `README.md`
- Продуктовый контекст сайта: `PRODUCT.md`
- Дизайн-токены маркетингового сайта: `DESIGN.md`
- Документация апгрейд-команд: `packages/twenty-server/docs/UPGRADE_COMMANDS.md`
- Публичная документация: <https://docs.twenty.com>
