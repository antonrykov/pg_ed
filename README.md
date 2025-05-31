# pg_ed
Проект для развертывания и изучения PostgreSQL на примере предложенной в книгах тестовой базы данных.
Запросы и тд выполнялись в psql. Тут будут приведены некоторые из них.

https://postgrespro.ru/education
# Книги
> PostgreSQL. Основы языка SQL
> 
> PostgreSQL. Профессиональный SQL

# Запуск
## Подымаем конейнер с бд
> docker compose up -d
## Подключаемся к контейнеру
> docker exec -it pg_ed-database-1 bash
## Подключиться к тестовой бд в psql
> psql app app -d demo
## Вставляем тестовый набор данных
> \i demo-small-20170815.sql
## Отобразить все таблицы 
> \d

# Индексы на основе выражений

> CREATE UNIQUE INDEX aircrafts_unique_model_key
> ON aircrafts_tmp ( lower( model ) );

![image](https://github.com/user-attachments/assets/6c70c2b7-d8c4-4a13-a7f1-df29fa3480d2)


# Частичные индексы
До

![image](https://github.com/user-attachments/assets/239dd1c5-6ebf-4879-9f97-d8fa5736c3d2)

Создаем частичный индекс

> CREATE INDEX bookings_book_date_part_key
> ON bookings ( book_date )
> WHERE total_amount > 1000000;

После

![image](https://github.com/user-attachments/assets/1940908f-73c3-4a46-aa58-dbdf226aabfa)

# Представления

На основе запроса создаем представление,

> CREATE VIEW seats_by_fare_cond AS
> 
> SELECT aircraft_code, fare_conditions, count( * )
> 
> FROM seats
> 
> GROUP BY aircraft_code, fare_conditions
> 
> ORDER BY aircraft_code, fare_conditions;

![image](https://github.com/user-attachments/assets/8ecf06a7-3ec3-4b52-a433-0a16c6eeb3ff)
