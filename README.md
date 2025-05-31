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


# Уровень изоляции Read Uncommitted

> CREATE TABLE aircrafts_tmp
> 
> AS SELECT * FROM aircrafts;

Апдейтим значние поля в транзакции, но не комитим её

![image](https://github.com/user-attachments/assets/2136d15c-48ee-4f05-a264-faf0aa541a62)

Во втором терминале проверям значние поля которое обновляли в 1 терминале

![image](https://github.com/user-attachments/assets/6d12b3ed-02c9-451d-88f1-7cbffaaec970)

Таким образом, вторая транзакция не видит изменение значения атрибута range, произведенное в первой — незафиксированной — транзакции. Это объясняется тем, что в PostgreSQL реализация уровня изоляции Read Uncommitted более строгая, чем того требует стандарт языка SQL. Фактически этот уровень тождественен уровню изоляции Read Committed.

# Уровень изоляции Read Committed

На первом терминале выполним следующие команды:

![image](https://github.com/user-attachments/assets/0f0cb64f-418f-4ff0-986f-e295f1cf553a)

На втором терминале вполняем:

![image](https://github.com/user-attachments/assets/549d4a4f-24ce-430d-b32a-2285e64e74ce)

Транзакция перешла в режим ожидания. Команда UPDATE в 1 терминале заблокировала строку в таблице aircrafts_tmp.

Выполним в 1 терминале команду COMMIT; и увидим что во 2 терминале команда заверщилась.

Как видно, были произведены оба изменения. Команда UPDATE во второй транзакции, получив возможность заблокировать строку после завершения первой транзакции и снятия ею блокировки с этой строки, перечитывает строку таблицы и потому обновляет строку, уже обновленную в только что зафиксированной транзакции. Таким образом, эффекта потерянных обновлений не возникает.

Для иллюстрации эффекта неповторяющегося чтения данных проведем совсем простой эксперимент также на двух терминалах. На первом терминале:

![image](https://github.com/user-attachments/assets/10e9aa5b-3382-456f-af54-39acfb7abee3)

Во втром терминале:

![image](https://github.com/user-attachments/assets/af1cd7ed-0f9a-468e-b42f-88c4580d48e6)

Повторим выборку в первой транзакции:

![image](https://github.com/user-attachments/assets/23bfdf00-210e-495c-a27a-3f55283b22c8)

Видим, что теперь получен другой результат, т. к. вторая транзакция завершилась в момент времени между двумя запросами. Таким образом, налицо эффект неповторяющегося чтения данных, который является допустимым на уровне изоляции Read Committed.

# Уровень изоляции Repeatable Read

Первый терминал:

![image](https://github.com/user-attachments/assets/407d63c1-0730-4cb2-91df-c8777911e972)

Второй терминал:

![image](https://github.com/user-attachments/assets/58a811f6-1076-4cca-aebb-5e1008784d8e)

Перейдем в первый терминал:

![image](https://github.com/user-attachments/assets/63faece0-7748-4907-9c20-a58e2a96185c)

На первом терминале ничего не изменилось: фантомные строки не видны, и также не видны изменения в уже существующих строках. Это объясняется тем, что снимок данных выполняется на момент начала выполнения первого запроса транзакции.
Завершим транзакцию в 1 терминале.

![image](https://github.com/user-attachments/assets/a1e92e60-a72e-49f0-b4f6-132ec2f2c6fd)

Как видим, одна строка добавлена, а значение атрибута range у самолета Airbus A320-200 стало на 100 больше, чем было.

# Уровень изоляции Serializable

Для проведения эксперимента создадим специальную таблицу, в которой будет всего два столбца: один — числовой, а второй — текстовый. Назовем эту таблицу modes.

> CREATE TABLE modes (num integer,mode text);

Добавим в таблцу две строки:

> INSERT INTO modes VALUES ( 1, 'LOW' ), ( 2, 'HIGH' );

Содерджимое таблицы имеет вид

![image](https://github.com/user-attachments/assets/b3c3118e-1ec0-4d09-afea-fa630f9a237b)

На первом терминале:

![image](https://github.com/user-attachments/assets/52db053d-8f1f-4ec5-a48f-6b6b1dc4ce33)

На втором терминале тоже начнем транзакцию и обновим другую строку из тех двух строк, которые были показаны выше.

![image](https://github.com/user-attachments/assets/c356fc1c-33ee-4023-a1f2-cb9d42aaa5e8)

Изменение, произведенное в первой транзакции, вторая транзакция не видит, поскольку на уровне изоляции Serializable каждая транзакция работает с тем снимком
базы данных, который был сделан непосредственно перед выполнением ее первого оператора. Поэтому обновляется только одна строка, та, в которой значение поля
mode было равно HIGH изначально.
Обратите внимание, что обе команды UPDATE были выполнены, ни одна из них не
ожидает завершения другой транзакции.

Посмотрим, что получилось в первой транзакции:

![image](https://github.com/user-attachments/assets/527b4247-011b-4055-a38c-f728de6fdfd2)

А во второй транзакции:

![image](https://github.com/user-attachments/assets/7c05d685-54eb-4637-9c93-d048dafc1821)

Завершим транзакции.

Во втором терминале видим: 

![image](https://github.com/user-attachments/assets/a18dba41-a193-4b66-8fa5-56220e11cf74)

Какое же изменение будет зафиксировано? То, которое сделала транзакция, первой выполнившая фиксацию изменений.

![image](https://github.com/user-attachments/assets/33a348e2-82bb-4737-8a12-dd120f6565c2)


