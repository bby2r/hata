# Restify MCP — примеры использования

MCP-сервер: `hatamatata-restify`
Транспорт: stdio через SSH до staging (`.mcp.json` в корне проекта).

Сейчас зарегистрирован один репозиторий — `s-e-o-texts` (модель `SEOText`, read-only) с кастомным геттером `resolve-seo`, который рендерит финальный title/h1/description страницы.

---

## Доступные MCP инструменты

| Инструмент | Что делает |
|------------|-----------|
| `global-search` | Поиск по всем репозиториям (по полям из `$search`) |
| `discover-repositories` | Список доступных репозиториев |
| `get-repository-operations` | Операции конкретного репо (index/show/getters/actions) |
| `get-operation-details` | Схема параметров одной операции |
| `execute-operation` | Запуск произвольной операции |
| `s-e-o-texts-index-tool` | Список/пагинация SEOText с фильтрами `page_type`, `country_id`, сортировкой и поиском |
| `s-e-o-texts-show-tool` | Одна запись по id |
| `s-e-o-texts-resolve-seo-getter-tool` | Рендер готового SEO (главная фича) |

---

## Примеры промптов для Claude

### Обзор структуры

> Через `hatamatata-restify` покажи, какие репозитории и операции у них доступны.

Claude вызовет `discover-repositories` → `get-repository-operations` и вернёт карту.

### Список шаблонов с фильтром

> Через `s-e-o-texts-index-tool` дай первые 5 шаблонов с `page_type=53` (город+тип+вид).

> Найди первые 10 шаблонов где `country_id=1`, отсортированные по id убыванию.

### Поиск

> Через `global-search` найди все записи где встречается "villa".

> Используя `s-e-o-texts-index-tool` с параметром `search=вилла` верни первые 3 совпадения.

### Одна запись

> Через `s-e-o-texts-show-tool` покажи SEOText id=86.

### Рендер готового SEO (самое полезное)

> Используй `s-e-o-texts-resolve-seo-getter-tool` для `deal=buy, country_slug=spain, type_label=Вилла`. Дай title, h1, description.

> Отрендери SEO для страницы аренды апартаментов в Барселоне с морским видом:
> `deal=rent, country_slug=spain, region_slug=catalonia, city_slug=barcelona, type_label=Апартаменты, view_label=Морской вид`

### Сравнение вариантов

> Отрендери SEO для вилл в Испании в двух вариантах: buy и rent. Покажи в таблице title/h1/description.

---

## Значения `page_type`

Берутся из `App\Enums\Pages`:

| Значение | Кейс | Описание |
|----------|------|----------|
| 1 | CountryPage | Страница страны |
| 2 | RegionPage | Страница региона |
| 3 | CityPage | Страница города |
| 4 | Main | Главная |
| 5 | CountryType | Страна + тип |
| 7 | CityType | Город + тип |
| 10 | CityView | Город + вид |
| 11 | BuyPage | Купить |
| 12 | RentPage | Арендовать |
| 13 | BuyType | Купить + тип |
| 15 | BuyTypeView | Купить + тип + вид |
| 53 | CityTypeView | Город + тип + вид |
| 56 | RentCityTypeView | Аренда: город + тип + вид |

Всего 60 значений — полный список в `app/Enums/Pages.php`.

---

## Параметры `resolve-seo` getter

Все опциональные:

| Параметр | Тип | Пример |
|----------|-----|--------|
| `deal` | `buy` или `rent` | `buy` (дефолт) |
| `country_slug` | string | `spain` |
| `region_slug` | string | `catalonia` |
| `city_slug` | string | `barcelona` |
| `type_label` | string | `Вилла`, `Апартаменты` |
| `view_label` | string | `Морской вид` |

Геттер резолвит слаги в модели, подставляет переменные в шаблоны через `FilterSeo` и возвращает `{title, h1, description, breadcrumbs_title}`.

---

## Авторизация

- **MCP** — работает без токенов. Ходит через stdio, напрямую через Eloquent.
- **Репозиторий** — read-only: `store`/`update`/`delete` вернут 403.
- **Write доступ** — открыть можно в `app/Restify/SEOTextRepository.php` (заменить `return false` в `authorizedToStore`/`Update`/`Delete`) + добавить валидацию в поля.

---

## Добавление новых репозиториев

```bash
php artisan restify:repository CountryRepository
```

Чтобы он автоматически появился в MCP:

```php
use Binaryk\LaravelRestify\MCP\Concerns\HasMcpTools;

class CountryRepository extends Repository
{
    use HasMcpTools;

    public function mcpAllowsShow(): bool { return true; }
    public function mcpAllowsGetters(): bool { return true; }

    public static array $search = ['name', 'slug'];
    public static array $match = ['parent_id' => 'integer'];
    // ...
}
```

После `config:clear` и рестарта MCP-сессии — tools вида `countries-index-tool`, `countries-show-tool` появятся автоматически.
