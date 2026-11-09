# ER-модель — кейс 3 «Онлайн-чаевые»

## Как заполнять

Дана одна сущность (Merchant) как образец и связи. Заполни остальные 5 сущностей,
их атрибуты и связи. Для каждой сущности укажи PK/FK, типы, обязательность и ограничения.

## 1\. Сущности (6)

|Сущность|Что хранит|Зачем|
|-|-|-|
|Merchant|merchant\_id, name, status, description|справочник мерчантов|
|Employee|employee\_id, merchant\_id, name, payment\_details|справочник официантов, привязанных к конкретному ресторану|
|QR|qr\_id, merchant\_id, employee\_id, receipt\_id, nonce, exp, signature|одноразовая ссылка на оплату с криптографической подписью|
|Payment|payment\_id, qr\_id, amount, tip, status, timestamp|факт оплаты (сумма чека + чаевые) и его текущий статус|
|Report|report\_id, merchant\_id, period, tips\_total|агрегаты для отчётности по чаевым за период|
|AuditEvent|event\_id, payment\_id, action, from\_status, to\_status, actor, at|журнал событий для разбора (логирование смены статусов)|

**Подсказки:**

* Employee — сотрудник (официант), привязан к мерчанту.
* QR — одноразовая ссылка на оплату с подписью.
* Payment — факт оплаты (чек + чаевые).
* Report — агрегаты для отчётности.
* AuditEvent — журнал событий для разбора.

## 2\. Атрибуты (заполни)

### 2.1. Merchant (ОБРАЗЕЦ — не трогать)

|Поле|Тип|PK/FK|Обяз.|Ограничение|
|-|-|-|-|-|
|merchant\_id|string|**PK**|да|`m\_NNN`|
|name|string|—|да|1–100|
|status|enum|—|да|ACTIVE/BLOCKED/PENDING|
|description|string|—|нет|0–200|

### 2.2. Employee (заполни)

|Поле|Тип|PK/FK|Обяз.|Ограничение|
|-|-|-|-|-|
|employee\_id|string|**PK**|да|`e\_NNN`|
|merchant\_id|string|**FK**|да|Ссылается на `Merchant.merchant_id`|
|name|string|—|да|1–100 символов|
|payment\_details|string|—|нет|Только маска `\*\*\*\*NNNN` (оригинал не хранится)|

**Подсказки:**

* `employee\_id` — `e\_NNN`.
* `merchant\_id` — FK → Merchant.
* `payment\_details` — только маска `\*\*\*\*NNNN`, не оригинал.

### 2.3. QR (заполни)

|Поле|Тип|PK/FK|Обяз.|Ограничение|
|-|-|-|-|-|
|qr\_id|string|**PK**|да|`qr\_NNN`|
|merchant\_id|string|**FK**|да|Ссылается на `Merchant.merchant_id`|
|employee\_id|string|**FK**|да|Ссылается на `Employee.employee_id`|
|receipt\_id|string|—|да|`r\_NNNN`, уникален для мерчанта|
|nonce|string|—|да|6 hex-символов (одноразовый)|
|exp|timestamp|—|да|ISO 8601 (срок действия 24 ч.)|
|signature|string|—|да|HMAC-SHA256, 16 hex-символов (не plain hash)|
**Подсказки:**

* `qr\_id` — `qr\_NNN`.
* `receipt\_id` — `r\_NNNN`, уникален для мерчанта.
* `nonce` — 6 hex, одноразовый.
* `exp` — ISO 8601, TTL 24 ч.
* `signature` — HMAC-SHA256, 16 hex.

### 2.4. Payment (заполни)

|Поле|Тип|PK/FK|Обяз.|Ограничение|
|-|-|-|-|-|
|payment\_id|string|**PK**|да|`p\_NNNN`|
|qr\_id|string|**FK**|да|Ссылается на `QR.qr_id`|
|amount|decimal|—|да|> 0, ≤ 1 000 000|
|tip|decimal|—|нет|≥ 0, ≤ amount × 0.5|
|status|enum|—|да|CREATED/PENDING/SUCCESS/DECLINED/CANCELED|
|timestamp|timestamp|—|да|ISO 8601 (время транзакции)|

**Подсказки:**

* `payment\_id` — `p\_NNNN`.
* `amount` — > 0, ≤ 1 000 000.
* `tip` — ≥ 0, ≤ amount × 0.5.
* `status` — CREATED/PENDING/SUCCESS/DECLINED/CANCELED.

### 2.5. Report (заполни)

|Поле|Тип|PK/FK|Обяз.|Ограничение|
|-|-|-|-|-|
|report\_id|string|**PK**|да|`rep\_NNNN`|
|merchant\_id|string|**FK**|да|Ссылается на `Merchant.merchant_id`|
|period|string|—|да|`YYYY-MM` (формат месяца)|
|tips\_total|decimal|—|да|≥ 0 (сумма чаевых за период)|

**Подсказки:**

* `report\_id` — `rep\_NNNN`.
* `period` — `YYYY-MM`.
* `tips\_total` — ≥ 0.

### 2.6. AuditEvent (заполни)

|Поле|Тип|PK/FK|Обяз.|Ограничение|
|-|-|-|-|-|
|event\_id|string|**PK**|да|`evt\_NNNN`|
|payment\_id|string|**FK**|да|Ссылается на `Payment.payment_id`|
|action|enum|—|да|payment_created / payment_confirmed / payment_declined / payment_canceled|
|from\_status|string|—|нет|Предыдущий статус платежа|
|to\_status|string|—|да|Новый статус платежа|
|actor|enum|—|да|system / client / officer|
|at|timestamp|—|да|ISO 8601 (время события)|

**Подсказки:**

* `event\_id` — `evt\_NNNN`.
* `action` — payment\_created / payment\_confirmed / payment\_declined / payment\_canceled.
* `actor` — system / client / officer.
* `at` — ISO 8601.

## 3\. Связи (заполни)

|Связь|Тип|Обоснование|
|-|-|-|
|Merchant → Employee|**1:N**|У одного мерчанта (ресторана) работает много официантов.|
|Merchant → QR|**1:N**|Один мерчант может сгенерировать много QR-кодов для разных чеков.|
|Employee → QR|**1:N**|Один официант может выставить много счетов (и сгенерировать много QR).|
|QR → Payment|**1:N**|По одному QR-коду может быть несколько попыток оплаты (например, если первый раз банк отклонил платеж, клиент пробует другую карту).|
|Merchant → Report|**1:N**|Для одного мерчанта формируется много отчетов (за разные месяцы/периоды).|
|Payment → AuditEvent|**1:N**|Каждый переход статуса платежа (создан, подтвержден, отменен) — это отдельное событие аудита.|

**Подсказки:**

* 1:N — у мерчанта много официантов, у официанта много QR и т.д.
* 1:N — по одному QR может быть несколько попыток платежа.
* 1:N — каждый переход статуса — отдельное событие аудита.

## 4\. Диаграмма (draw.io)

Нарисуйте 6 прямоугольников (сущности) с полями PK/FK и соедините стрелками
с подписями `1` и `N`. Экспортируйте в PNG и вставьте в приложение (раздел A.2).

## 5\. Критерий приёмки

* \[ ] 6 сущностей описаны
* \[ ] Для каждой сущности указаны PK/FK, типы и ограничения
* \[ ] Связи описаны с типом (1:N, 1:1) и обоснованием
* \[ ] `payment\_details` — только маска `\*\*\*\*NNNN`
* \[ ] `signature` — HMAC-SHA256, не plain hash
* \[ ] Диаграмма в draw.io с PK/FK и связями

