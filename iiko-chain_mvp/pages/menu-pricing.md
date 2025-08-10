### Меню и цены — MVP

Функции:
- Меню/позиции, базовые рецепты (read-only из HO)
- `PriceZone` + `PriceList`, массовые обновления

```mermaid
flowchart LR
  Menu --> Publish
  PriceZone --> PriceList --> Publish
```

