# Progress NW Catalog API

OpenAPI 3.0 спецификация для каталога [progress-nw.ru](https://progress-nw.ru/catalog).

## 📖 Просмотр документации
- **Онлайн**: [https://AndreeMe.github.io/progress-api-docs/](https://AndreeMe.github.io/progress-api-docs/)
- **Исходник**: [swagger.yaml](./swagger.yaml)

## ⚙️ Для разработчика
| Параметр | Значение |
|----------|----------|
| Формат | JSON, UTF-8 |
| Аутентификация | Заголовок `X-API-Key` |
| Rate Limit | 100 запросов/мин |
| База | `https://progress-nw.ru/api/v1` |

## 🚀 Быстрый старт
```bash
curl -H "X-API-Key: YOUR_KEY" \
  https://progress-nw.ru/api/v1/products?limit=5
