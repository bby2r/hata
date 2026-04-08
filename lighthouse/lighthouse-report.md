# Lighthouse Audit Report — hatamatata.ru / hatamatata.com

**Дата:** 2026-04-08

---

## Результаты до исправлений

| Категория | .ru Mobile | .ru Desktop | .com Mobile | .com Desktop |
|---|---|---|---|---|
| Performance | 84 | 59 | 68 | 83 |
| Accessibility | 85 | 85 | 85 | 85 |
| Best Practices | 77 | 77 | 77 | 77 |
| SEO | 100 | 100 | 100 | 100 |

### Метрики до исправлений

| Метрика | .ru Mobile | .ru Desktop | .com Mobile | .com Desktop |
|---|---|---|---|---|
| FCP | 2.1s | 0.7s | 1.5s | 0.7s |
| LCP | 3.0s | 2.3s | 8.1s | 2.2s |
| TBT | 280ms | 690ms | 130ms | 10ms |
| CLS | 0.024 | 0.003 | 0.024 | 0.003 |
| Speed Index | 4.8s | 2.0s | 6.6s | 2.4s |
| TTI | 12.8s | 2.7s | 11.4s | 3.1s |

---

## Что было исправлено

### Коммит 1: e6e7c1e34 — Accessibility и базовая оптимизация

#### Accessibility

- **Buttons without accessible name** — добавлены `aria-label` на кнопку WhatsApp, гамбургер-меню, swiper-навигацию (prev/next)
- **Links without discernible name** — добавлены `aria-label` на все социальные ссылки в footer (VK, OK, Facebook, Instagram, Telegram, Twitter)
- **Heading order** — исправлена иерархия заголовков на главной: h4/h5/h6 заменены на h3/h4 для соблюдения последовательности (h1 -> h2 -> h3 -> h4). Затронуты: agencies, popular_posts, citizenship_for_investment, reviews, customer_choice, properties_by_filter, write_seller_card, contact_us
- **Color contrast** — цвет `text-brand-gray-4` изменён с `#9EA5B2` (contrast ratio ~3.5:1) на `#6B7280` (contrast ratio ~5.3:1) для соответствия WCAG AA

#### Performance

- **Unsized images** — добавлены явные `width` и `height` на изображения: irina.png, house.svg, house-2.svg, 66.svg, 67.svg, 9.svg
- **LCP image preload** — добавлен `<link rel="preload">` для hero-баннера на главной странице

### Коммит 2: 5f7586d4f — Performance оптимизация

#### 1. Отложена загрузка аналитики (+3s после window.load)

- `counters-head.blade.php` — GTM/Metrica/Ads обёрнуты в `setTimeout` с 3s задержкой
- `footer.blade.php` — body-счётчики аналогично отложены
- `errors/404.blade.php` — Yandex Metrica и Google Analytics отложены
- **Эффект:** убирает аналитику из критического пути загрузки, снижает TBT и TTI

#### 2. Code split app.js (-53% размера)

- Swiper — динамический `import()`, загружается только если `.mySwiper` есть на странице
- Flowbite — динамический `import()` (lazy)
- **Результат:** app.js 386 KB -> 180 KB (gzip: 117 KB -> 64 KB)
- Swiper выделен в отдельный чанк (79 KB), загружается по запросу
- Flowbite выделен в отдельный чанк (130 KB), загружается асинхронно

#### 3. FontAwesome subset (-55% CSS)

- Создан `subset.css` (~2 KB) вместо `all.css` (209 KB) — только 19 используемых иконок
- Убраны устаревшие форматы шрифтов (eot, ttf, svg) — оставлены только woff2/woff
- Добавлен `font-display: swap` вместо `auto` — устраняет FOIT (Flash of Invisible Text)
- **Результат:** app.css 314 KB -> 141 KB (gzip: 59 KB -> 24 KB)

#### 4. Build target es2020 — убраны legacy polyfills

- Добавлен `build.target: 'es2020'` в `vite.config.js`
- Устраняет генерацию polyfills для устаревших браузеров (~11 KB)

### Коммит 3: a28abde08 — Self-hosted Google Fonts

- **Libre Baskerville** (400, 400i, 700) — скачан woff2, используется в layouts: pages, agreement, personal-data
- **Poppins** (400-700, normal + italic) — скачан woff2 (8 файлов), используется в тех же layouts
- **Inter** (variable 400-800, latin + cyrillic) — скачан woff2 (2 файла), используется на странице dubai-developers
- Создан `google-fonts-local.css` с локальными `@font-face` + `font-display: swap`
- Убраны все внешние ссылки на `fonts.googleapis.com` и `fonts.gstatic.com`

### Суммарный эффект

| Ресурс | До | После | Экономия |
|---|---|---|---|
| app.js | 386 KB (gzip: 117 KB) | 180 KB (gzip: 64 KB) | **-53%** |
| app.css | 314 KB (gzip: 59 KB) | 141 KB (gzip: 24 KB) | **-55%** |
| FontAwesome CSS | 209 KB | ~2 KB | **-99%** |
| Шрифты FA (форматы) | eot+woff2+woff+ttf+svg | woff2+woff | **-60% файлов** |
| Google Fonts | 3 внешних запроса | Локально через Vite | **-3 запроса** |
| Аналитика (TBT impact) | Синхронная | Отложена на 3s | **~0ms TBT** |

---

## Результаты после исправлений (.com — задеплоено)

| Категория | Mobile (до) | Mobile (после) | Desktop (до) | Desktop (после) |
|---|---|---|---|---|
| **Performance** | 68 | 65 | 83 | **98 (+15)** |
| **Accessibility** | 85 | **96 (+11)** | 85 | **94 (+9)** |
| **Best Practices** | 77 | **100 (+23)** | 77 | **100 (+23)** |
| **SEO** | 100 | 100 | 100 | 100 |

### Метрики после исправлений (.com)

| Метрика | Mobile (до) | Mobile (после) | Desktop (до) | Desktop (после) |
|---|---|---|---|---|
| FCP | 1.5s | 3.0s | 0.7s | **0.5s** |
| LCP | 8.1s | 8.6s | 2.2s | **0.6s (-73%)** |
| TBT | 130ms | **60ms (-54%)** | 10ms | **0ms** |
| CLS | 0.024 | **0** | 0.003 | 0.003 |
| Speed Index | 6.6s | 5.7s | 2.4s | **1.5s (-38%)** |
| TTI | 11.4s | 9.0s | 3.1s | **0.6s (-81%)** |

> Mobile Performance не вырос из-за LCP (8.6s) — hero-изображение тяжёлое на медленных соединениях. Desktop показал значительный рост по всем метрикам.

---

## Не исправлено (требует DevOps или внешних решений)

| Проблема | Влияние | Причина |
|---|---|---|
| Inefficient cache policy | Medium | Настраивается на уровне nginx/CDN — заголовки `Cache-Control` для статики |
| Third-party cookies | Medium | GTM, Yandex Metrica, Google Ads. Только server-side tagging решит |
| Chrome DevTools issues | Low | Deprecation warnings от сторонних скриптов |

### Серверная инфраструктура

| Проблема | Решение | Кто решает |
|---|---|---|
| Cache-Control headers | Добавить в nginx: `expires 1y` для `/build/assets/`, `expires 7d` для `/images/`, `/svg/`, `/assets/` | DevOps |
| Gzip/Brotli compression | Включить `gzip_static on` и `brotli on` в nginx для JS/CSS/SVG | DevOps |
| CDN | Вынести статику на CDN (CloudFlare, AWS CloudFront) для уменьшения TTFB | Инфраструктура |
