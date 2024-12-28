# Отчет по выполнению задания

## Описание задачи

- Работа с датасетом Airbnb Open Data: [Kaggle Dataset](https://www.kaggle.com/datasets/arianazmoudeh/airbnbopendata)
- Загрузка данных в виртуальную машину (VM)
- Создание внешней и внутренней таблиц в базе данных
- Выполнение SQL-запросов для анализа данных

---

## Выполненные шаги

### Шаг 1: Подготовка окружения
#### Создание директории
```bash
VM$ mkdir team-73
```

#### Копирование датасета на виртуальную машину
На локальной машине:
```bash
scp -P 22 Airbnb_Open_Data.csv user@91.185.85.179:/home/user/team-73
```

#### Запуск `gpfdist` на VM
```bash
VM$ gpfdist -d . -p 8073 &
```

---

### Шаг 2: Создание внешней таблицы

Создание внешней таблицы `team73`:
```sql
CREATE EXTERNAL TABLE team73 (
    id TEXT,
    name TEXT,
    host_id TEXT,
    host_identity_verified TEXT,
    host_name TEXT,
    neighbourhood_group TEXT,
    neighbourhood TEXT,
    lat TEXT,
    long TEXT,
    country TEXT,
    service_fee TEXT,
    minimum_nights TEXT,
    number_of_reviews TEXT,
    last_review TEXT,
    reviews_per_month TEXT,
    review_rate_number TEXT,
    calculated_host_listings_count TEXT,
    availability_365 TEXT,
    house_rules TEXT,
    license TEXT
)
LOCATION ('gpfdist://127.0.0.1:8073/Airbnb_Open_Data.csv')
FORMAT 'csv' (HEADER);
```

---

### Шаг 3: Создание внутренней таблицы

Создание внутренней таблицы `team73_internal` и преобразование типов данных:
```sql
CREATE TABLE team73_internal AS
SELECT
    CAST(TRIM(id) AS INTEGER) AS id,
    NAME,
    CAST(TRIM(host_id) AS BIGINT) AS host_id,
    host_identity_verified,
    host_name,
    neighbourhood_group,
    neighbourhood,
    CAST(TRIM(lat) AS FLOAT) AS lat,
    CAST(TRIM(long) AS FLOAT) AS long,
    country,
    country_code,
    instant_bookable,
    cancellation_policy,
    room_type,
    CAST(TRIM(construction_year) AS INTEGER) AS construction_year,
    CAST(REPLACE(TRIM(REPLACE(price, '$', '')), ',', '') AS FLOAT) AS price,
    CAST(REPLACE(TRIM(REPLACE(service_fee, '$', '')), ',', '') AS FLOAT) AS service_fee,
    CAST(REPLACE(TRIM(minimum_nights), ',', '') AS INTEGER) AS minimum_nights,
    CAST(REPLACE(TRIM(number_of_reviews), ',', '') AS INTEGER) AS number_of_reviews,
    TO_DATE(TRIM(last_review), 'MM/DD/YYYY') AS last_review,
    CAST(REPLACE(TRIM(reviews_per_month), ',', '') AS FLOAT) AS reviews_per_month,
    CAST(REPLACE(TRIM(review_rate_number), ',', '') AS FLOAT) AS review_rate_number,
    CAST(REPLACE(TRIM(calculated_host_listings_count), ',', '') AS INTEGER) AS calculated_host_listings_count,
    CAST(REPLACE(TRIM(availability_365), ',', '') AS INTEGER) AS availability_365,
    house_rules,
    license
FROM team73;
```

---

### Шаг 4: Проверка данных

#### Просмотр одной строки из внешней таблицы:
```sql
SELECT * FROM team73 LIMIT 1;
```

**Вывод:**
```
   id    |                name                |   host_id   | host_identity_verified | host_name | neighbourhood_group | neighbourhood |   lat    |   long    |    country    | country_code | instant_bookable | cancellation_policy |  room_type   
| construction_year | price | service_fee | minimum_nights | number_of_reviews | last_review | reviews_per_month | review_rate_number | calculated_host_listings_count | availability_365 |                                     house_rules          
                            | license
---------+------------------------------------+-------------+------------------------+-----------+---------------------+---------------+----------+-----------+---------------+--------------+------------------+---------------------+--------------
+-------------------+-------+-------------+----------------+-------------------+-------------+-------------------+--------------------+--------------------------------+------------------+----------------------------------------------------------
----------------------------+---------
 1001254 | Clean & quiet apt home by the park | 80014485718 | unconfirmed            | Madaline  | Brooklyn            | Kensington    | 40.64749 | -73.97237 | United States | US           | FALSE            | strict              | Private room
| 2020              | $966  | $193        | 10             | 9                 | 10/19/2021  | 0.21              | 4                  | 6                              | 286              | Clean up and treat the home the way you'd like your home
to be treated.  No smoking. |
(1 row)
```

---

### Шаг 5: Анализ данных

#### Вычисление средней стоимости аренды:
```sql
idp=> SELECT AVG(price) FROM team73_internal;
        avg        
-------------------
 625.2935360325152
(1 row)
```

---

## Заключение

Все шаги выполнены успешно:
1. Датасет был загружен на VM.
2. Созданы внешняя и внутренняя таблицы.
3. Данные преобразованы в нужный формат.
4. Выполнены запросы для проверки данных и анализа.

