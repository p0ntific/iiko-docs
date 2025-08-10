### Складские документы — MVP

Функции:
- Списание по рецептам при оплате
- Передача данных в iikoOffice

Backend:
- POST /stock/consume {orderId} — списание по рецептам
- Webhook → iikoOffice `{storeId, saleItems, consumption}`

Acceptance:
- Консистентность суммарного расхода vs продажи (±0.5% толеранс); ретраи 3× при недоступности бэка.

