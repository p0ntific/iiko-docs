### Кухонный экран — MVP

Функции:
- Очередь блюд, статусы, таймеры

User stories:
- Как повар, вижу список тикетов с блюдамии по приоритету, могу менять статус: В работе → Готово.
- Как шеф, фильтрую по станциям (гриль/салаты/бар) и вижу SLA блюд.

UI/UX:
- Список заказов слева (номер, стол/чек, таймер), карточка блюд по станциям справа.
- Горячие клавиши: Start (S), Done (D), Recall (R).

Frontend:
- WebSocket подписка `orders/kds`; локальная очередь событий; офлайн-буфер на 50 событий.

Backend:
- GET /kds/orders?station=... — список активных блюд
- POST /kds/orders/{id}/start | /done | /recall
- Push канал: ws /kds/stream (orderTicket events)

Модель данных (упрощенно): `KdsTicket{id, orderId, course, station, itemName, qty, status, startedAt, dueAt}`

Acceptance:
- P95 обновления UI < 1s, корректное упорядочивание при out-of-order событиях, протоколирование статусов.

