# Установка и использование PySpark на AWS через pip

## i) Установка PySpark
### Убедитесь, что установлен Python 3:
Если Python 3 не установлен:
```bash
VM1$ sudo apt-get install -y python3 python3-pip
```

### Установка PySpark:
```bash
VM1$ pip install pyspark
```

## ii) Запуск PySpark
```bash
VM1$ pyspark --master yarn \
        --deploy-mode client \
        --conf spark.hadoop.fs.defaultFS=hdfs://<IP_VM1>:9000
```
**Примечание:** Замените `<IP_VM1>` на ваш IP-адрес или используйте `localhost`.

### Пример загрузки и обработки данных:
```python
>>> df_raw = spark.read \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .csv("/user/hadoop/data/employees.csv")

>>> df_raw.show(5)
>>> df_raw.printSchema()
```

## iii) Преобразования данных в PySpark
```python
from pyspark.sql.functions import col, lit, avg, max as max_, count, concat_ws

# 1) Фильтр по условию: выбираем сотрудников с зарплатой >= 50000
df_filtered = df_raw.filter(col("salary") >= 50000)

# 2) Добавляем новое поле (withColumn): перевод зарплаты в тысячи
df_with_col = df_filtered.withColumn("salary_k", col("salary") / 1000)

# 3) Переименуем столбец "name" в "employee_name"
df_renamed = df_with_col.withColumnRenamed("name", "employee_name")

# 4) Группировка (GroupBy + Agg): считаем среднюю, максимальную зарплату и кол-во сотрудников
df_agg = df_renamed.groupBy("department") \
    .agg(
        avg("salary").alias("avg_salary"),
        max_("salary").alias("max_salary"),
        count("*").alias("count_employees")
    )

# 5) Объединение (Union) с другим DataFrame
# Для примера создадим новый DataFrame с одной колонкой "summary"
df_agg_str = df_agg.select(
    concat_ws(" | ",
        col("department"),
        col("avg_salary"),
        col("max_salary"),
        col("count_employees")
    ).alias("summary")
)

df_extra = spark.createDataFrame([("Some extra info",), ("Spark Transformation Demo",)], ["summary"])
df_union = df_agg_str.union(df_extra)

df_union.show(truncate=False)
```

## iv) Сохранение данных в партиционированную таблицу Hive

### Проверка доступности базы данных:
```python
spark.sql("SHOW DATABASES").show()
```
Если видно в выводе `company`, двигаемся дальше.

### Переключение на базу `company` и сохранение:
```python
spark.sql("USE company")

df_renamed.write \
    .mode("overwrite") \
    .format("hive") \
    .partitionBy("department") \
    .saveAsTable("employees_spark_part")
```

## v) Работа с Hive через Beeline
```bash
^D + beeline -u "jdbc:hive2://localhost:10000" -n hadoop -p 12qwaszx
```

### Выполнение запросов в Hive:
```sql
USE company;
SHOW TABLES;
DESCRIBE FORMATTED employees_spark_part; -- чтобы проверить и увидеть партиции
```

