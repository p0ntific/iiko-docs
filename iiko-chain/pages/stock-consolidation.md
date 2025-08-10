### Консолидация остатков и продаж

**Назначение**: единое окно по остаткам и продажам сети, мониторинг дефицита.

**Функции**
- Централизованная витрина `SalesAggregate`, `StockSnapshot`
- Критические остатки и автоуведомления
- Экспорт для BI/финансов

### Схема

```mermaid
flowchart TB
  Stores[Точки] --> Sales[Продажи]
  Stores --> Stock[Остатки]
  Sales --> DWH[Витрина]
  Stock --> DWH
  DWH --> Reports[Отчёты/BI]
```

### Роли
- Analytics, Finance, Region Manager

