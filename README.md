# Schema Forge

Маленький статический сайт для генерации **JSON Schema** (Draft-07) или **Avro Schema** из произвольного JSON. Всё работает в браузере — ни бэкенда, ни сборки, ни зависимостей.

![Schema Forge](https://img.shields.io/badge/deps-0-brightgreen) ![Static](https://img.shields.io/badge/static-site-blue) ![License](https://img.shields.io/badge/license-MIT-lightgrey)

## Возможности

- Переключатель в шапке: **JSON Schema ⇄ Avro**
- Две колонки: слева — JSON, справа — сгенерированная схема
- Живая валидация JSON с подсветкой ошибок
- Подсветка синтаксиса в результате
- Кнопка копирования схемы в буфер обмена
- Готовый пример одним кликом
- Один файл `index.html`, ~19 КБ

## Как это выглядит

```
┌───────────────────────────────────────────────────────────┐
│  Schema Forge        [ JSON Schema | Avro ]               │
├─────────────────────────────┬─────────────────────────────┤
│  {                          │  {                          │
│    "id": 42,                │    "$schema": "...",        │
│    "name": "Alice",         │    "type": "object",        │
│    "tags": ["a", "b"]       │    "properties": { ... }    │
│  }                          │  }                          │
└─────────────────────────────┴─────────────────────────────┘
```

## Как работает вывод типов

### JSON Schema (Draft-07)

| JSON value           | Выводится как            |
|----------------------|--------------------------|
| `"text"`             | `{ "type": "string" }`   |
| `42`                 | `{ "type": "integer" }`  |
| `3.14`               | `{ "type": "number" }`   |
| `true` / `false`     | `{ "type": "boolean" }`  |
| `null`               | `{ "type": "null" }`     |
| `[...]`              | `{ "type": "array", "items": ... }` |
| `{...}`              | `{ "type": "object", "properties": {...}, "required": [...] }` |

Массивы с разнородными элементами сворачиваются в `anyOf`. Все ключи объекта считаются `required` (проще потом убрать ненужные, чем дописывать).

### Avro Schema

| JSON value           | Avro-тип                 |
|----------------------|--------------------------|
| `"text"`             | `string`                 |
| `42`                 | `long`                   |
| `3.14`               | `double`                 |
| `true` / `false`     | `boolean`                |
| `null`               | `["null", "string"]` с `"default": null` (nullable) |
| `[...]`              | `{ "type": "array", "items": ... }` |
| `{...}`              | `{ "type": "record", "name": "...", "fields": [...] }` |

Имена `record` генерируются из ключей в `PascalCase`. Массивы с разнородными элементами превращаются в Avro union.
