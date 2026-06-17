# Практическая работа №1: Продвинутые возможности PostgreSQL

**Выполнила:** Магомадова Фирюза Юсуповна  
**Группа:** ЦИБ-241  
**Дата:** 04/06/2026  
**Инструмент:** Yandex DataLens + PostgreSQL 

---

## Выбранные задания

| № | Блок | Тема | Задание |
|---|------|------|---------|
| 1 | **А** | Анализ времени и дат | Дни недели продаж |
| 2 | **Б** | Геопространственный анализ | Ближайший дилер для клиентов из NY |
| 3 | **В** | Сложные типы (JSONB) | Поиск по атрибутам в JSONB |

---

## Цель работы

Научиться применять продвинутые возможности PostgreSQL для анализа данных:
- Анализ временных рядов с `DATE_TRUNC`, `EXTRACT`
- Геопространственный анализ с расчетом расстояний
- Работа с JSONB структурами и поиск по атрибутам
- Визуализация результатов в Yandex DataLens

---

## Задание 1. Дни недели продаж (Блок А)

### Условие

Определить, в какой день недели (понедельник, вторник и т.д.) совершается наибольшее количество продаж. Вывести день недели, количество транзакций и общую сумму продаж.

### SQL-код

```sql
SELECT 
    CASE EXTRACT(DOW FROM sales_transaction_date)
        WHEN 0 THEN 'Sunday'
        WHEN 1 THEN 'Monday'
        WHEN 2 THEN 'Tuesday'
        WHEN 3 THEN 'Wednesday'
        WHEN 4 THEN 'Thursday'
        WHEN 5 THEN 'Friday'
        WHEN 6 THEN 'Saturday'
    END AS day_of_week,
    COUNT(*) AS number_of_sales,
    ROUND(SUM(sales_amount)::numeric, 2) AS total_sales_amount
FROM sales
GROUP BY EXTRACT(DOW FROM sales_transaction_date)
ORDER BY number_of_sales DESC;
```
***(скрины успешного выполнения прикреплены и подписаны)***

### Вывод
Наибольшее количество продаж совершается в Friday (пятницу) — 3 транзакции. Общая сумма продаж в этот день составляет 85 000.00. Это может быть связано с тем, что покупатели чаще совершают крупные покупки перед выходными.

---
## Задание 2. Ближайший дилер для клиентов из NY (Блок Б)
### 📌 Условие
Для каждого клиента из штата NY (state = 'NY') найти ближайший дилерский центр и расстояние до него в милях.

***Примечание: Расстояние рассчитано по формуле Гаверсина (Haversine), так как расширение earthdistance было недоступно на учебном сервере.***

### SQL-код
```
sql
WITH distances AS (
    SELECT 
        c.customer_id,
        CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
        CONCAT(d.city, ', ', d.state) AS dealership_name,
        3959 * ACOS(
            COS(RADIANS(c.latitude)) * COS(RADIANS(d.latitude)) *
            COS(RADIANS(d.longitude) - RADIANS(c.longitude)) +
            SIN(RADIANS(c.latitude)) * SIN(RADIANS(d.latitude))
        ) AS distance_miles,
        ROW_NUMBER() OVER (
            PARTITION BY c.customer_id 
            ORDER BY 3959 * ACOS(
                COS(RADIANS(c.latitude)) * COS(RADIANS(d.latitude)) *
                COS(RADIANS(d.longitude) - RADIANS(c.longitude)) +
                SIN(RADIANS(c.latitude)) * SIN(RADIANS(d.latitude))
            )
        ) AS rn
    FROM 
        customers c 
    CROSS JOIN 
        dealerships d
    WHERE 
        c.state = 'NY'
)
SELECT 
    customer_id,
    customer_name,
    dealership_name,
    ROUND(distance_miles::numeric, 2) AS distance_miles
FROM distances
WHERE rn = 1
ORDER BY distance_miles;
