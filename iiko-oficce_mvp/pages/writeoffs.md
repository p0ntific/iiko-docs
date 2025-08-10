### Списания — MVP

Функции:
- Типы причин (браки/порча)
- Списание FEFO/FIFO

Backend API:
- POST /writeoffs {storeId, reason, items:[{sku, qty, batchId?}]}

Валидации: запрет отрицательных остатков; автоподбор партий FEFO если batchId не задан.

