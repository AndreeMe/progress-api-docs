# Техническое задание: OpenAPI спецификация Progress NW Catalog API

## 1. Общие сведения

**Проект:** Progress NW Catalog API  
**Версия спецификации:** 1.0.0  
**Формат:** OpenAPI 3.0.3  
**Репозиторий:** https://github.com/AndreeMe/progress-api-docs  
**Публикация:** https://andreeme.github.io/progress-api-docs/

## 2. Назначение

Спецификация описывает REST API для получения каталога вентиляционного оборудования с поддержкой:
- Синхронизации с учётными системами (1С)
- Векторного поиска (RAG — Retrieval Augmented Generation)
- Фильтрации по техническим характеристикам
- Получения метаданных категорий и схем характеристик

## 3. Требования к спецификации

### 3.1. Структура

```
swagger.yaml
├── openapi: 3.0.3
├── info
│   ├── title: Progress NW Catalog API
│   ├── description: API для получения товарных данных с поддержкой векторного поиска (RAG)
│   ├── version: 1.0.0
│   └── contact.email: api@progress-nw.ru
├── servers
│   └── url: https://progress-nw.ru/api/v1
├── security: ApiKeyAuth
├── components
│   ├── securitySchemes
│   │   └── ApiKeyAuth (apiKey, header, X-API-Key)
│   ├── schemas
│   │   ├── ProductStatus
│   │   ├── CategoryRef
│   │   ├── SpecificationItem
│   │   ├── MediaItem
│   │   ├── DocumentItem
│   │   ├── AccessoryItem
│   │   ├── ProductForRAG
│   │   ├── ProductListResponse
│   │   └── Error
│   └── responses
│       ├── Unauthorized
│       ├── Forbidden
│       ├── NotFound
│       ├── RateLimited
│       └── DefaultError
├── paths
│   ├── /products (GET)
│   ├── /products/{product_id} (GET)
│   ├── /categories (GET)
│   └── /specs/schema/{category_slug} (GET)
└── tags
    ├── Products
    ├── Categories
    └── Metadata
```

### 3.2. Требования к схемам

#### ProductForRAG

| Поле | Тип | Обяз. | Ограничения | Описание |
|------|-----|-------|-------------|----------|
| `id` | string | Да | `^[a-zA-Z0-9\-_]+$`, max 64 | Уникальный идентификатор |
| `slug` | string | Нет | `^[a-z0-9\-_/]+$`, max 255 | ЧПУ-идентификатор |
| `name` | string | Да | `^[^<>]+$`, max 255 | Наименование товара |
| `category` | CategoryRef | Да | — | Категория товара |
| `price` | number | Да | min 0, max 10 000 000 | Цена в рублях |
| `currency` | string | Нет | `^[A-Z]{3}$`, enum: [RUB] | Валюта |
| `stock` | integer | Нет | min 0, max 100 000 | Остаток на складе |
| `searchable_text` | string | Да | max 4000 | Текст для RAG-векторизации |
| `specifications` | SpecificationItem[] | Да | 0–100 элементов | Технические характеристики |
| `spec_schema_ref` | string (uri) | Нет | `^https?://`, max 512 | URL JSON Schema характеристик |
| `media` | object | Нет | images: 0–20, documents: 0–10 | Изображения и документы |
| `accessories` | AccessoryItem[] | Нет | 0–50 элементов | Комплектующие |
| `status` | ProductStatus | Да | enum: [active, archived] | Статус товара |
| `seo` | object | Нет | title: max 70, meta_description: max 160 | SEO-метаданные |
| `updated_at` | string (date-time) | Да | max 30 | Дата последнего обновления |
| `created_at` | string (date-time) | Нет | max 30 | Дата создания |

#### CategoryRef

| Поле | Тип | Обяз. | Ограничения | Описание |
|------|-----|-------|-------------|----------|
| `id` | string | Да | `^[a-zA-Z0-9\-_]+$`, max 64 | Идентификатор категории |
| `name` | string | Да | `^[^<>]+$`, max 100 | Наименование |
| `slug` | string | Да | `^[a-z0-9\-]+$`, max 100 | URL-идентификатор |
| `level` | integer | Нет | 0–5 | Уровень вложенности |

#### SpecificationItem

| Поле | Тип | Обяз. | Ограничения | Описание |
|------|-----|-------|-------------|----------|
| `key` | string | Да | `^[a-z_]+$`, max 50 | Унифицированный ключ |
| `label` | string | Да | `^[^<>]+$`, max 100 | Человекочитаемое название |
| `value` | string | Да | `^[^<>]+$`, max 255 | Значение (строка) |
| `value_num` | number | Нет | float, ±9 999 999 | Числовое значение |
| `unit` | string | Нет | `^[a-zA-Z0-9/\-°%]+$`, max 20 | Единица измерения |
| `group` | string | Нет | `^[a-z_]+$`, max 50 | Группа характеристик |
| `is_filterable` | boolean | Нет | default: false | Доступна для фильтрации |

### 3.3. Требования к эндпоинтам

#### GET /products

**Назначение:** Получение списка товаров с фильтрацией и пагинацией.

**Параметры:**

| Имя | Тип | Обяз. | По умолчанию | Описание |
|-----|-----|-------|--------------|----------|
| `category` | string | Нет | — | Фильтр по slug категории |
| `status` | enum | Нет | — | Фильтр по статусу |
| `search` | string | Нет | — | Текстовый поиск |
| `spec_filter` | string | Нет | — | `key:operator:value` |
| `updated_from` | date-time | Нет | — | Синхронизация 1С |
| `page` | integer | Нет | 1 | Номер страницы |
| `limit` | integer | Нет | 50 | Размер страницы (макс: 100) |

**Ответы:**
- `200` — ProductListResponse
- `400` — DefaultError
- `401` — Unauthorized
- `403` — Forbidden
- `406` — DefaultError
- `429` — RateLimited

#### GET /products/{product_id}

**Назначение:** Получение детальной информации о товаре.

**Параметры:**
- `product_id` (path, required) — `^[a-zA-Z0-9\-_]+$`, max 64

**Ответы:**
- `200` — ProductForRAG
- `401` — Unauthorized
- `403` — Forbidden
- `404` — NotFound
- `406` — DefaultError
- `429` — RateLimited

#### GET /categories

**Назначение:** Получение дерева категорий.

**Ответы:**
- `200` — CategoryRef[] (0–500 элементов)
- `401` — Unauthorized
- `406` — DefaultError
- `429` — RateLimited

#### GET /specs/schema/{category_slug}

**Назначение:** Получение схемы допустимых характеристик для категории.

**Параметры:**
- `category_slug` (path, required) — `^[a-z0-9\-]+$`, max 100

**Ответы:**
- `200` — объект с `category` и `allowed_specs[]`
- `404` — NotFound

### 3.4. Security Quality Gates

Все строковые поля должны иметь `pattern`:

| Паттерн | Применение |
|---------|------------|
| `^[a-z0-9\-]+$` | slug, category_slug |
| `^[a-zA-Z0-9\-_]+$` | id, product_id |
| `^[a-z_]+$` | key, group характеристик |
| `^[^<>]+$` | name, label, value, search, title, description |
| `^https?://` | url, spec_schema_ref |
| `^[A-Z]{3}$` | currency |
| `^[A-Z_]+$` | error code |
| `^[a-zA-Z0-9/\-°%]+$` | unit (единицы измерения) |
| `^[^:]+:[^:]+:[^<>]+$` | spec_filter (key:operator:value) |

### 3.5. Требования к валидации

- Файл должен проходить `swagger-cli validate swagger.yaml`
- Все `$ref` должны быть корректными
- Все `required` поля должны присутствовать в схемах
- Все `enum` значения должны быть согласованы

## 4. Публикация на GitHub

### 4.1. Структура репозитория

```
progress-api-docs/
├── swagger.yaml      # Исходная спецификация
├── swagger.json      # Скомпилированная версия (JSON)
├── index.html        # Страница документации
├── README.md         # Описание проекта
└── PR.md             # Описание Pull Request
```

### 4.2. GitHub Pages

- **Источник:** ветка `main`, корневая директория
- **URL:** https://andreeme.github.io/progress-api-docs/
- **Рендеринг:** Redoc Standalone через CDN

### 4.3. Процесс публикации

1. Внести изменения в `swagger.yaml`
2. Проверить валидность: `swagger-cli validate swagger.yaml`
3. Скомпилировать в JSON: `swagger-cli bundle swagger.yaml -o swagger.json -t json`
4. Закоммитить: `git add swagger.yaml swagger.json && git commit -m "..."`
5. Отправить: `git push`
6. GitHub Pages обновится автоматически через 1–2 минуты

### 4.4. index.html

Страница загружает `swagger.json` и рендерит через Redoc:

```html
<script src="https://cdn.jsdelivr.net/npm/redoc@latest/bundles/redoc.standalone.js"></script>
<script>
  fetch('swagger.json')
    .then(res => res.json())
    .then(spec => Redoc.init(spec, {}, document.getElementById('redoc-container')));
</script>
```

## 5. Версионирование

- Версия API указывается в `info.version`
- URL сервера содержит версию: `https://progress-nw.ru/api/v1`
- При изменении мажорной версии обновлять URL сервера

## 6. Контактная информация

- **Репозиторий:** https://github.com/AndreeMe/progress-api-docs
- **Документация:** https://andreeme.github.io/progress-api-docs/
