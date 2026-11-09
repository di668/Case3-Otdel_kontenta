# Механизм эмуляции платежа — кейс 3 (заполни и сверь с эталоном)

## 1. Состояния платежа


* CREATED - платёж создан на сервере: QR валидирован, суммы рассчитаны, запись зафиксирована, но клиент ещё не подтвердил оплату.
* PENDING —  клиент подтвердил, запрос передан в эмулятор шлюза, результат ещё не получен (обработка / ожидание ответа / таймаут).
* SUCCESS — эмулятор вернул код 0–7: оплата прошла, чаевые зачислены официанту, платёж финальный.
* DECLINED — эмулятор вернул код 8: шлюз отклонил операцию (CARD_DECLINED), деньги не списаны, попытка финальная.
* CANCELED — платёж отменён клиентом/системой до подтверждения (из CREATED) или по правилу отмены; деньги не списаны.

## 2. Матрица переходов


|Из состояния|В состояние|Условие|
|-|-|-|
|CREATED|PENDING|Клиент нажал «Подтвердить оплату», QR прошёл все проверки Q-1…Q-6, платёж записан|
|PENDING|SUCCESS|Эмулятор вернул h ∈ 0…7|
|PENDING|DECLINED|Эмулятор вернул h = 8 (CARD_DECLINED)|
|PENDING|PENDING|Эмулятор вернул h = 9 (TIMEOUT) — статус не меняется, клиенту «проверьте через минуту»|
|CREATED|CANCELED|Клиент нажал «Отменить» до подтверждения (или истёк TTL платежа)|
|PENDING|CANCELED|Отмена разрешена только если шлюз ещё не вернул финальный статус (по правилу P-4 — в общем случае запрещена, 409 CANCEL_NOT_ALLOWED; допускается только системный откат по таймауту)|
|SUCCESS|CANCELED|Запрещено — 409 CANCEL_NOT_ALLOWED (деньги уже списаны, нужен возврат отдельным процессом)|

## 3. Шаги обработки платежа

1. Валидация запроса (POST /api/v1/tips): проверка обязательных полей (qr_id, receipt_id, base_amount, tip_pct, idempotency_key), типов и диапазонов → при ошибке 422 VALIDATION_ERROR.

2. Проверка QR: загрузка qr_links.csv по qr_id, проверка Q-1 signature (HMAC-SHA256), Q-2 exp ≥ now, Q-3 nonce не использован, Q-4 merchant.status = ACTIVE, Q-5 receipt_id совпадает, Q-6 tip_pct ≤ 50 → при ошибке 410/422/409/403, платёж не создаётся.

3. Расчёт суммы: tip_amount = round(base × tip_pct/100, 2), amount_total = base + tip; проверки P-1 (base ≤ 0 или > 1 000 000 → AMOUNT_OUT_OF_RANGE), P-3 (amount_total > 2 000 000 → AMOUNT_OUT_OF_RANGE).

4. Создание платежа: проверка idempotency_key (P-2): если ключ уже есть — вернуть существующий payment_id, HTTP 200, новую строку не писать; иначе запись в payments.csv со статусом CREATED, генерация payment_id = p_NNNN.

5. Подтверждение: POST /api/v1/payments/{id}/confirm → переход CREATED → PENDING, вызов emulate_gateway(qr_id, nonce) — h = md5(qr_id+nonce) mod 10.

6. Результат: h ∈ 0…7 → SUCCESS (200); h = 8 → DECLINED + CARD_DECLINED (402); h = 9 → PENDING, TIMEOUT (200/202, retryable).

7. Отмена: POST /api/v1/payments/{id}/cancel → из CREATED разрешена (CANCELED, 200), из PENDING и SUCCESS → 409 CANCEL_NOT_ALLOWED; каждая попытка пишется в аудит.

8. Аудит: на каждом шаге запись события AuditEvent (event_id, payment_id, qr_id, action, from_status, to_status, amount_total, actor, at); в логи — только маска реквизитов.

## 4. Формулы


* tip_amount = round(base_amount × tip_pct / 100, 2)
* amount_total = base_amount + tip_amount
* h = int(md5((qr_id + nonce).encode()).hexdigest(), 16) mod 10
* SIGNATURE = HMAC-SHA256(SECRET, "merchant_id|employee_id|receipt_id|nonce|exp").hexdigest()[:16]

## 5. Пример расчёта (qr_001)


* base_amount = 637.00
* tip_pct = 5
* tip_amount = 637.00 × 5 / 100 = 31.85
* amount_total = 637.00 + 31.85 = 668.85
* h = int(md5("qr_001" + "c82561").hexdigest(), 16) mod 10 = 3 (0…7)
* статус = SUCCESS

## 6. Пример DECLINED (qr_003)

* base_amount = 911.00
* tip_pct = 15
* tip_amount = 911.00 × 15 / 100 = 136.65
* amount_total = 911.00 + 136.65 = 1047.65
* h = int(md5("qr_003" + "4443ae").hexdigest(), 16) mod 10 = 8
* статус = DECLINED, error_code = CARD_DECLINED

## 7. Обработка ошибок


|Ситуация|error_code|HTTP|retryable|
|-|-|-|-|
|QR signature не совпала|INVALID_QR|422|нет|
|QR истёк|QR_EXPIRED|410|да (новый QR)|
|nonce использован|QR_USED|409|нет|
|merchant BLOCKED|MERCHANT_BLOCKED|403|нет|
|tip_pct > 50|TIP_OUT_OF_RANGE|422|нет|
|amount_total > 2 000 000|AMOUNT_OUT_OF_RANGE|422|нет|
|тот же idempotency_key|— (возврат существующего)|200|да (безопасно)|
|эмулятор вернул 8|CARD_DECLINED|402|да (другой способ)|
|эмулятор вернул 9|GATEWAY_TIMEOUT|202|да|
|отмена из PENDING|CANCEL_NOT_ALLOWED|409|нет|
|отмена из SUCCESS|CANCEL_NOT_ALLOWED|409|нет|

## 8. Идемпотентность

При повторном запросе с тем же idempotency_key сервер находит существующий платёж и возвращает его payment_id и статус, не создавая новую строку в payments.csv. В файле остаётся одна строка на ключ. HTTP-код — 200 OK (не 201), тело — тот же Payment. Повторная обработка шлюза не выполняется, аудит фиксирует событие payment_duplicate_requested с actor=system.

## 9. Аудит

1. event_id — evt_NNNN
2. payment_id — p_NNNN
3. qr_id — qr_NNN
4. action — payment_created / payment_confirmed / payment_declined / payment_canceled / payment_duplicate_requested
5. from_status — CREATED / PENDING / —
6. to_status — PENDING / SUCCESS / DECLINED / CANCELED
7. actor — system / client / officer
8. at — ISO 8601 (timestamp)


