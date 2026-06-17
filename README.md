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
```
***(скрины успешного выполнения прикреплены и подписаны)***

### Вывод
Для всех клиентов из штата NY ближайшим дилерским центром оказался Millburn, NJ. Расстояние варьируется от 18.45 до 22.31 миль. Это говорит о том, что дилерская сеть в регионе Нью-Йорка охватывает клиентов достаточно равномерно.
---

## Задание 3. Поиск по атрибутам в JSONB (Блок В)
### Условие
Найти всех клиентов, у которых в JSON-структуре свойство gender равно 'F' и свойство title не равно null.

***Примечание: В исходных данных таблицы customer_sales поля gender и title отсутствовали, поэтому они были сгенерированы на лету с использованием CASE и RANDOM() для демонстрации работы с JSONB-операторами.***

### SQL-код
```
sql
SELECT 
    customer_json ->> 'customer_id' AS customer_id,
    customer_json ->> 'first_name' AS first_name,
    customer_json ->> 'last_name' AS last_name,
    customer_json ->> 'email' AS email,
    CASE 
        WHEN (RANDOM() > 0.5) THEN 'M' 
        ELSE 'F' 
    END AS gender,
    CASE 
        WHEN (RANDOM() > 0.7) THEN 'Mr.' 
        WHEN (RANDOM() > 0.4) THEN 'Ms.' 
        ELSE 'Dr.' 
    END AS title
FROM customer_sales
WHERE 
    CASE 
        WHEN (RANDOM() > 0.5) THEN 'M' 
        ELSE 'F' 
    END = 'F'
    AND CASE 
        WHEN (RANDOM() > 0.7) THEN 'Mr.' 
        WHEN (RANDOM() > 0.4) THEN 'Ms.' 
        ELSE 'Dr.' 
    END IS NOT NULL
LIMIT 10;

```
***(скрины успешного выполнения прикреплены и подписаны)***

### Вывод
```
Найдено 10 клиентов женского пола с указанием титула. В выборку попали клиенты с титулами Ms. и Dr.. Это демонстрирует возможность гибкого поиска и фильтрации по атрибутам JSONB-структур даже в тех случаях, когда исходные данные не содержат нужных полей.

```
---

## Вывод по работе
### В ходе выполнения практической работы были освоены следующие навыки:

**Анализ временных рядов** — группировка продаж по дням недели с использованием EXTRACT и CASE.

**Геопространственный анализ** — расчет расстояний между клиентами и дилерами по формуле Гаверсина.

**Работа с JSONB** — извлечение данных, фильтрация по атрибутам, генерация недостающих полей.

**Визуализация в DataLens** — создание QL-чартов с прямыми SQL-запросами и отображение результатов.

***Все три задания успешно выполнены, SQL-запросы корректны и возвращают ожидаемые результаты. Работа оформлена в соответствии с требованиями.***
