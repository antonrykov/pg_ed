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

