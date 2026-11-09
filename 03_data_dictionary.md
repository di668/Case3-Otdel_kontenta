# Data dictionary — кейс 3



## Образец merchants (не трогать)
|Файл|Поле|Тип|Обяз.|Ограничение|Пример|Код ошибки|
|-|-|-|-|-|-|-|
|merchants|merchant\_id|string|да|m\_NNN|m\_001|—|
|merchants|name|string|да|1–100 символов|Вкус-Сервис Тверская|—|
|merchants|status|enum|да|ACTIVE/BLOCKED/PENDING|ACTIVE|MERCHANT\_BLOCKED|
|merchants|description|string|нет|0–200|Ресторан сети|—|



## Образец payments (не трогать)


|Файл|Поле|Тип|Обяз.|Ограничение|Пример|Код ошибки|
|-|-|-|-|-|-|-|
|payments|payment\_id|string|да|p\_NNNN|p\_0001|—|
|payments|amount|decimal|да|> 0, ≤ 1 000 000|637.00|AMOUNT\_OUT\_OF\_RANGE|
|payments|tip|decimal|да|≥ 0, ≤ amount × 0.5|63.70|TIP\_OUT\_OF\_RANGE|
|payments|status|enum|да|CREATED/PENDING/SUCCESS/DECLINED/CANCELED|SUCCESS|—|



## Таблица — employees



|Поле|Тип|Обяз.|Ограничение|Пример|Код ошибки|
|-|-|-|-|-|-|
|employee\_id|string|да|e_NNN; уникальный идентификатор сотрудника|e_001|-|
|merchant\_id|string|да|FK -> merchants; формат m_NNN|m_001|-|
|name|string|да|1-100 символов|Иванов И.|-|
|payment\_details|string|да|только маска ****NNNN; последние 4 цифры реквизитов|****1001|-|

## Таблица — qr\_links



|Поле|Тип|Обяз.|Ограничение|Пример|Код ошибки|
|-|-|-|-|-|-|
|qr\_id|string|да|qr_NNN|qr_001|-|
|merchant\_id|string|да|FK -> merchnts; формат m_NNN|m_001|MERCHATN_BLOCKED|
|employee\_id|string|да|FK -> employees; формат e_NNN; сотрудник должен относиться к указанному мерчанту|e_001|-|
|receipt\_id|string|да|r_NNNN; уникален для одного мерчанта|r_1001|RECEIPT_MISMATCH|
|nonce|string|да|Ровно 6 hex символов; одноразовый|с82561|QR_USED|
|exp|datetime|да|ISO 8601; срок действия QR - 24 часа|2025-11-01T23:00Z|QR_EXPIRED|
|signature|string|да|HMAC-SHA256; первые 16 символов hex пердставления подписи|68d0f361ac38c12b|INVALID_QR|



## Критерий приёмки

- \[ ] Все 4 файла описаны (22 колонки)

- \[ ] Для каждого поля указан тип и ограничение

- \[ ] Для полей с валидацией — код ошибки из 08\_errors.md

- \[ ] Пример взят из реального CSV

- \[ ] payment\_details — только маска (\*\*\*\*NNNN)



## Подсказки

- payment\_details — это реквизиты для выплат, не для оплаты.

- nonce — одноразовый, 6 hex.

- exp — время, после которого QR недействителен (обычно 24 ч).

- signature — HMAC-SHA256, 16 hex.

