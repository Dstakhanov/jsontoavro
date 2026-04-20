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

## Быстрый старт (локально)

Просто открой `index.html` в браузере:

```bash
git clone https://github.com/USERNAME/schema-forge.git
cd schema-forge
xdg-open index.html   # Linux
open index.html       # macOS
```

Или подними любым HTTP-сервером:

```bash
python3 -m http.server 8080
# → http://localhost:8080
```

## Деплой на Debian 12 с nginx

```bash
# установка
sudo apt update
sudo apt install -y nginx

# файлы
sudo mkdir -p /var/www/schema-forge
sudo cp index.html /var/www/schema-forge/
sudo chown -R www-data:www-data /var/www/schema-forge

# конфиг
sudo tee /etc/nginx/sites-available/schema-forge >/dev/null <<'EOF'
server {
    listen 80 default_server;
    server_name _;
    root /var/www/schema-forge;
    index index.html;

    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }
}
EOF

# активация
sudo ln -sf /etc/nginx/sites-available/schema-forge /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
```

Открывай `http://<ip-сервера>/`.

### HTTPS с Let's Encrypt

Нужен домен, указывающий на сервер (свой или бесплатный через [DuckDNS](https://www.duckdns.org)).

```bash
sudo apt install -y certbot python3-certbot-nginx

# заменить _ на свой домен в конфиге
sudo sed -i 's/server_name _;/server_name example.com;/' \
  /etc/nginx/sites-available/schema-forge
sudo systemctl reload nginx

# получить сертификат (certbot сам дополнит конфиг)
sudo certbot --nginx -d example.com

# проверить, что автопродление работает
sudo certbot renew --dry-run
```

## Деплой на GitHub Pages

1. Запушь репозиторий на GitHub
2. **Settings → Pages → Source: Deploy from a branch → main / root**
3. Через пару минут сайт будет на `https://USERNAME.github.io/schema-forge/`

## Структура проекта

```
schema-forge/
├── index.html   ← весь сайт: HTML + CSS + JS в одном файле
└── README.md
```

Никаких `package.json`, `node_modules`, сборщиков и бэкенда. Хочешь что-то поменять — правь `index.html` и обновляй страницу.

## Технологии

- Чистый HTML / CSS / vanilla JS (ES2019+)
- Google Fonts: Fraunces, IBM Plex Sans, JetBrains Mono
- Без фреймворков, без сборки, без зависимостей

## Ограничения

- Вывод типов работает по одному примеру JSON — не собирает статистику по массиву разных документов
- Строковые форматы (`date`, `email`, `uri`) не определяются автоматически — только `"type": "string"`
- Avro namespace по умолчанию — `com.example`, при необходимости поменять в коде
- Для Google Fonts нужен интернет при первой загрузке страницы (шрифты кэшируются браузером)

## Лицензия

MIT
