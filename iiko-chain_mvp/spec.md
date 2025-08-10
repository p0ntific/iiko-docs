### iikoChain MVP — Техническое задание (по стримам)

Цель: централизованное управление меню и ценами, публикации на точки, сбор продаж/остатков. Релиз в 2 спринта по 2 недели.

### Область MVP (In)
- Организации: бренды/регионы/точки с наследованием
- Меню (позиции) и зоны цен, прайс-листы
- Публикации (меню/цены) с волнами и мониторингом статусов
- Отчёты: продажи по точкам/брендам, состояние публикаций, остатки (снэпшоты)
- Роли: HO Admin, Brand/Region Manager, Franchise (own stores)

Out: SCM/DC, сложные акции, продвинутый ETL.

---
### Дизайн (UX/UI)
- Навигация: разделы `Орг. структура`, `Меню и цены`, `Публикации`, `Отчёты`, `Настройки`
- Табличные списки с: поиск, фильтры, пагинация (25/50/100), массовые операции
- Форма редактирования зоны цен: таблица цен (SKU × Цена × Валюта × Дата начала)
- Мастер публикации: выбор артефактов (меню/прайс-лист), выбор волн, предпросмотр затронутых точек
- Состояние публикаций: канбан (Черновик/Утв./В волне/Доставлено) + список точек со статусами
- Цветовая маркировка статусов: draft(gray), approved(blue), in-wave(amber), delivered(green), failed(red)

Acceptance (UX):
- Все формы адаптивны ≥1280px, контраст WCAG AA, сорт-иконки фокусируются с клавиатуры
- Таблицы >10k строк — виртуальный скролл

---
### Frontend
- Технологии: React 18 + TypeScript, React Query, Zustand, Vite, ESLint/Prettier, Jest + RTL
- Роутинг: `/org`, `/menu`, `/prices`, `/publications`, `/reports`, `/settings`
- Состояния и кэш: React Query с TTL 5 мин; optimistic updates для цен и публикаций
- Internationalization: ru-RU, en-US (i18next)

Компоненты/экраны:
- OrgTree: древовидный список брендов/регионов/точек; CRUD (модалки)
- MenuGrid: список позиций (sku, наимен., категория, активность); просмотр рецепта read-only
- PriceZoneEditor: CRUD зон, таблица цен с массовым редактированием; импорт CSV (sku;price)
- PublicationWizard: пошагово (1) выбрать артефакты (меню/цены), (2) выбрать точки/волны, (3) подтвердить
- PublicationMonitor: таблица точек с состояниями, перезапуск неуспешных доставок
- Reports: граф «Продажи по брендам», таблица «Статус публикаций», «Остатки»

Acceptance (FE):
- 95% критических путей покрыты тестами (логика редактора цен и публикаций)
- Все формы валидируются на клиенте и показывают ошибки API

---
### Backend
- Технологии: .NET 8 (или Node.js NestJS) + PostgreSQL 15, Redis, REST + Webhook, OpenAPI 3.1
- Микросервисы (минимально): Identity, Catalog (menu), Pricing, Publication, Sync, Reporting
- Асинхронная доставка публикаций: RabbitMQ (topic `chain.publications`) + ретраи (DLQ)

API (основные)

```http
POST /auth/login
200 { accessToken, refreshToken }

GET  /org/tree
POST /org/brand { name }
POST /org/region { brandId, name }
POST /org/store { regionId, name, code }

GET  /menu/items?brandId&regionId&storeId

GET  /prices/zones
POST /prices/zones { name, stores:[id] }
PUT  /prices/zones/{id}
POST /prices/zones/{id}/pricelist: multipart CSV (sku;price)
GET  /prices/zones/{id}/pricelist

POST /publications { type: "menu|price", artifactId, stores:[id], waveAt }
GET  /publications/{id}
POST /publications/{id}/approve
POST /publications/{id}/dispatch  # пуш в шину

POST /sync/webhook  # точки отвечают: { storeId, publicationId, status }

GET  /reports/sales?from&to&brandId
GET  /reports/publications/status?from&to
GET  /reports/stock?date
```

Модель данных (упрощённо)

```sql
table brand(id uuid pk, name text)
table region(id uuid pk, brand_id uuid fk, name text)
table store(id uuid pk, region_id uuid fk, name text, code text unique)
table menu_item(id uuid pk, sku text unique, name text, category text)
table price_zone(id uuid pk, name text)
table price_zone_store(price_zone_id uuid fk, store_id uuid fk, pk(price_zone_id, store_id))
table price_item(id uuid pk, price_zone_id uuid fk, sku text, price numeric, start_date date)
table publication(id uuid pk, type text, artifact_id uuid, status text, wave_at timestamptz)
table publication_store(publication_id uuid fk, store_id uuid fk, status text, updated_at timestamptz)
table sales_agg(date date, store_id uuid, sku text, qty numeric, revenue numeric)
table stock_snapshot(date date, store_id uuid, sku text, qty numeric)
```

Бизнес-правила:
- Нельзя публиковать пустой прайс-лист / меню без активных позиций
- `publication_store.status` только из набора: draft, approved, queued, delivering, delivered, failed
- Повторная доставка разрешена только для failed

НФТ (NFR):
- SLA пиковая нагрузка: 200 RPS чтение, 50 RPS запись; p95<300ms
- Ретраи публикаций: 3 попытки (эксп. бэкофф 1s/5s/25s)
- Аудит изменений для зон цен и публикаций (в журнал)

Acceptance (BE):
- OpenAPI авто-генерируется, e2e тест публикации в песочнице (mock store)
- Idempotency-Key на POST публикаций и импорте прайс-листов

---
### Порядок релиза / спринты
- Спринт 1: Org + Menu list (read-only) + Price zones CRUD + импорт/просмотр прайса + отчёт «Продажи»
- Спринт 2: Публикации + мониторинг + вебхуки от точек + отчёт «Статус публикаций» + остатки

Definition of Done
- Дизайн: макеты Figma, спецификация состояний/ошибок
- FE: тесты ≥70% statements в критических модулях, Lighthouse ≥90
- BE: e2e тест сценария публикации, алерты по DLQ, миграции БД

