## Отчётность — кейс 3

## 1\. Отчёт по мерчанту (ОБРАЗЕЦ)

**Endpoint:** `GET /api/v1/reports/tips?merchant\_id=m\_001\&from=2025-11-01\&to=2025-11-30`

**Метрики:**

|Метрика|Формула|Пример|
|-|-|-|
|tips\_total|sum(tip)|550.00|
|payments\_count|count(payments)|4|
|avg\_tip\_pct|avg(tip / amount × 100)|10.0|
|success\_rate|count(SUCCESS) / count(\*)|0.75|

**Пример ответа:**

```json
{
  "merchant\_id": "m\_001",
  "period": "2025-11",
  "tips\_total": 550.00,
  "payments\_count": 4,
  "avg\_tip\_pct": 10.0,
  "success\_rate": 0.75
}
```

## 2\. Отчёт по официанту (заполни)

**Endpoint:** `GET /api/v1/reports/tips?employee\_id=e\_001\&from=2025-11-01\&to=2025-11-30`

**Метрики (заполни формулы):**

|Метрика|Формула|Пример|
|-|-|-|
|tips_total| `SUM(tip)` за период | `83.60` |
|payments_count| `COUNT(payment_id)` за период | `2` |
|avg_tip_amount| `SUM(tip) / COUNT(payment_id)` | `41.80` |
|top_day| День (дата), в который была максимальная сумма чаевых | `"2025-11-03"` |

**Пример ответа (заполни):**

```json
{
  "employee\_id": "e_001",
  "period": "2025-11-01/2025-11-04",
  "tips\_total": 83.60,
  "payments\_count": 2,
  "avg\_tip_amount": 41.80,
  "top\_day": "2025-11-03"
}
```

## 3\. Отчёт по дням (заполни)

**Endpoint:** `GET /api/v1/reports/tips/daily?merchant\_id=m\_001\&from=2025-11-01\&to=2025-11-30`

**Метрики:**

|Метрика|Формула|Пример|
|-|-|-|
|date|—|2025-11-01|
|tips\_total|sum(tip) за день|200.00|
|payments\_count|count за день|2|
|avg\_tip\_pct|avg(tip/amount) за день|8.5|

**Пример ответа (заполни):**

```json
{
  "merchant\_id": "m\_001",
  "daily": \[
    {"date": "2025-11-01", "tips\_total": 246.15, "payments\_count": 3, "avg\_tip\_pct": 10.5},
    {"date": "2025-11-02", "tips\_total": 365.30, "payments\_count": 3, "avg\_tip\_pct": 11.2}
  ]
}
```

## 4\. Критерий приёмки

* \[ ] Три разреза: мерчант, официант, по дням
* \[ ] Метрики с формулами
* \[ ] Примеры ответов в JSON
* \[ ] Данные сходятся с `payments.csv`
* \[ ] Указаны endpoint'ы

```


