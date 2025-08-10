### Остатки — MVP

Функции:
- Просмотр остатков по складам/партиям
- Движение по документам (drill-down)

Backend API:
- GET /stock/balances?storeId
- GET /stock/movements?sku&from&to

НФТ: отклик < 400ms при 10k строк, серверная пагинация.

