# Restify API & MCP — примеры использования

База: **https://hata-new.doxer.work**
Префикс API: `/api/restify`
MCP-сервер: `hatamatata-restify` (регистрируется в `.mcp.json`)

Сейчас зарегистрирован один репозиторий — `s-e-o-texts` (модель `SEOText`, read-only) с кастомным геттером `resolve-seo`, который рендерит финальный title/h1/description страницы.

---

## 1. HTTP-запросы

### 1.1. Список всех SEO шаблонов (пагинация)

```bash
curl 'https://hata-new.doxer.work/api/restify/s-e-o-texts?perPage=5'
```

Ответ:
```json
{
  "meta": { "current_page": 1, "last_page": 6, "per_page": 5, "total": 30 },
  "data": [
    {
      "id": "86",
      "type": "s_e_o_texts",
      "attributes": {
        "id": 86,
        "page_type": 60,
        "country_id": null,
        "title": "Foreign real estate…",
        "h1": "…",
        "description": "…"
      }
    }
  ]
}
```

### 1.2. Фильтр по типу страницы (page_type)

Значения `page_type` берутся из `App\Enums\Pages`:

| Значение | Кейс |
|----------|------|
| 1 | CountryPage |
| 2 | RegionPage |
| 3 | CityPage |
| 7 | CityType |
| 10 | CityView |
| 13 | BuyType |
| 15 | BuyTypeView |
| 53 | CityTypeView |
| 56 | RentCityTypeView |
| 60 | Main |

```bash
# Все шаблоны для "город + тип + вид" (CityTypeView)
curl 'https://hata-new.doxer.work/api/restify/s-e-o-texts?page_type=53&perPage=5'
```

### 1.3. Фильтр по стране

```bash
curl 'https://hata-new.doxer.work/api/restify/s-e-o-texts?country_id=1'
```

### 1.4. Комбинация фильтров

```bash
curl 'https://hata-new.doxer.work/api/restify/s-e-o-texts?page_type=53&country_id=1'
```

### 1.5. Поиск по тексту

Поиск идёт по полям `title`, `h1`, `description`, `route_example`:

```bash
curl 'https://hata-new.doxer.work/api/restify/s-e-o-texts?search=вилла'
```

### 1.6. Сортировка

```bash
# по убыванию id
curl 'https://hata-new.doxer.work/api/restify/s-e-o-texts?sort=-id&perPage=3'

# по page_type возрастанию
curl 'https://hata-new.doxer.work/api/restify/s-e-o-texts?sort=page_type'
```

### 1.7. Получить одну запись

```bash
curl 'https://hata-new.doxer.work/api/restify/s-e-o-texts/86'
```

### 1.8. Геттер `resolve-seo` — рендер готового SEO

Шаблоны содержат плейсхолдеры `VAR_COUNTRY`, `VAR_CITY`, `VAR_TYPE` и т.д. Геттер подставляет их и возвращает готовый текст для конкретной страницы.

Параметры (все опциональные):
- `deal` — `buy` или `rent` (по умолчанию `buy`)
- `country_slug`, `region_slug`, `city_slug`
- `type_label` — название типа объекта (`Вилла`, `Апартаменты`)
- `view_label` — название вида (`Морской вид`)

```bash
# Купить виллы с морским видом в Каталонии
curl 'https://hata-new.doxer.work/api/restify/s-e-o-texts/getters/resolve-seo?deal=buy&country_slug=spain&region_slug=catalonia&type_label=Вилла&view_label=Морской%20вид'
```

Ответ:
```json
{
  "deal": "buy",
  "country": "spain",
  "region": "catalonia",
  "city": null,
  "type": "Вилла",
  "view": "Морской вид",
  "title": "Купить вилла in Каталонияс видом морской вид, цены от 13 989 $ ┋ ХатаМатата",
  "h1": "Купить вилла in Каталония  с видом морской вид",
  "description": "Продажа вилла за границей морской вид ┋ in Каталония | Цены от 13 989 $ до 8 043 896 $ 176 объектов",
  "breadcrumbs_title": null
}
```

### 1.9. Список всех доступных геттеров

```bash
curl 'https://hata-new.doxer.work/api/restify/s-e-o-texts/getters'
```

---

## 2. MCP через Claude Code

Если `.mcp.json` в корне содержит `hatamatata-restify`, Claude увидит инструменты:

- `mcp__hatamatata-restify__discover-repositories`
- `mcp__hatamatata-restify__get-repository-operations`
- `mcp__hatamatata-restify__get-operation-details`
- `mcp__hatamatata-restify__execute-operation`
- `mcp__hatamatata-restify__global-search`

### Примеры промптов

> Через `hatamatata-restify` покажи, какие репозитории доступны.

> Найди в `s-e-o-texts` первые 5 шаблонов с `page_type=53`.

> Найди все SEO-шаблоны, где title содержит «вилла», отсортируй по id убыванию.

> Покажи SEOText id=86.

> Вызови геттер `resolve-seo` на `s-e-o-texts` с параметрами `deal=buy`, `country_slug=spain`, `region_slug=catalonia`, `type_label=Вилла`, `view_label=Морской вид`.

> Через `global-search` найди упоминания «Барселона» во всех репозиториях.

---

## 3. Авторизация и запись

`SEOTextRepository` сейчас настроен read-only:

| Операция | Разрешено |
|----------|-----------|
| index / show / getters | ✅ |
| store / update / delete | ❌ |

Чтобы разрешить запись — снять `return false` в `authorizedToStore` / `authorizedToUpdate` / `authorizedToDelete` в `app/Restify/SEOTextRepository.php` и добавить валидационные правила в поля.

---

## 4. Добавление новых репозиториев

```bash
php artisan restify:repository CountryRepository
```

Это создаст `app/Restify/CountryRepository.php`. После определения `$search`, `$match`, `$sort` и `fields()` он автоматически появится в MCP как набор tools без дополнительного кода.
