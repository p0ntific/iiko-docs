### Инвентаризация — MVP

Функции:
- Полная/выборочная, ввод факта, отклонения
- Завершение с корректировкой остатков

Backend API:
- POST /inventory/sessions {storeId, type:"full|partial"}
- POST /inventory/sessions/{id}/counts {items:[{sku, qty}]}
- POST /inventory/sessions/{id}/close

Acceptance: нельзя закрыть при незаполненных позициях; печатный отчёт расхождений.

