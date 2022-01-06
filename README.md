# ДЗ 12
## Секционировать большую таблицу из демо базы flights

скачал демобазу по адресу https://postgrespro.com/education/demodb

распаковал архив и поправил создание таблицы Flights следующим образом:
```
CREATE TABLE flights (
    flight_id integer NOT NULL,
    flight_no character(6) NOT NULL,
    scheduled_departure timestamp with time zone NOT NULL,
    scheduled_arrival timestamp with time zone NOT NULL,
    departure_airport character(3) NOT NULL,
    arrival_airport character(3) NOT NULL,
    status character varying(20) NOT NULL,
    aircraft_code character(3) NOT NULL,
    actual_departure timestamp with time zone,
    actual_arrival timestamp with time zone,
    CONSTRAINT flights_check CHECK ((scheduled_arrival > scheduled_departure)),
    CONSTRAINT flights_check1 CHECK (((actual_arrival IS NULL) OR ((actual_departure IS NOT NULL) AND (actual_arrival IS NOT NULL) AND (actual_arrival > actual_departure)))),
    CONSTRAINT flights_status_check CHECK (((status)::text = ANY (ARRAY[('On Time'::character varying)::text, ('Delayed'::character varying)::text, ('Departed'::character varying)::text, ('Arrived'::character varying)::text, ('Scheduled'::character varying)::text, ('Cancelled'::character varying)::text])))
)
partition by range (scheduled_departure);
```

Далее создал секционирование на 2016, 2017 года и по умолчанию:
```
CREATE TABLE flights_2016 partition of flights for values from ('2016-01-01') to ('2017-01-01');
CREATE TABLE flights_2017 partition of flights for values from ('2017-01-01') to ('2018-01-01');
CREATE TABLE flights_default partition of flights default;
```

и накатил всю таблицу коммандой `sudo psql -h 127.0.0.1 -U postgres -d partition_db -a -f demo-big-en-20170815.sql`

после чего всё встало почти как надо, только констрейнты с основными и внешними ключами, которые затрагивают данную таблицу слетели. можно расширить ключ таблицы на данную колонку, но тогда надо будет еще и триггер вешать, который будет отрезать одинаковые айди с разными временами, что бы сохранить старую логику добавления и так же просмотреть остальные зависимости таблиц и сделать аналогичные правки

Возможно если сделать секционирование по hash на 2-4 таблицы - такой проблемы не будет, но это не проверенная теория
