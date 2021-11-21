# Занятие №10: Разворачиваем и настраиваем БД с большими данными

1. Выбрать одну из СУБД
2. Загрузить в неё данные (10 Гб)  
**Выполнение**: Для данного пункта был выбран Google Cloud BigQuery. Необходимые данные там уже содержатся.
3. Сравнить скорость выполнения запросов на PosgreSQL и выбранной СУБД  
**Выполнение**: Для сравнения использовал запрос:
```
В PostgreSQL:
select count(*) from taxi_trips;

В BigQuery:
SELECT count(*) FROM `bigquery-public-data.chicago_taxi_trips.taxi_trips`
```
Время выполнения запроса в BigQuery - менее секунды, в PostgreSQL - 17 минут 52 секунды.  
Далее решил еще попробовать более слоный запрос:
```
В PostgreSQL:
SELECT extract(month from trip_start_timestamp) as start_month,
        payment_type as payment_type,
        sum(trip_total) as summary
    FROM taxi_trips
    group by 1,2;

В BigQuery:
SELECT extract(month from trip_start_timestamp) as start_month,
        payment_type as payment_type,
        sum(trip_total) as summary
    FROM `bigquery-public-data.chicago_taxi_trips.taxi_trips`
    group by 1,2;
```
Время выполнения запроса в BigQuery - 2.2 секунды, в PostgreSQL - 17 минут 42 секунды. 
4. Описать что и как делали и с какими проблемами столкнулись  
**Выполнение**:
- Экспортировал из BigQuery данные датасета new_york_taxi_trips.tlc_yellow_trips_2017 в формате CSV
- Закинул их на машину с PostgreSQL и залил в БД через COPY:
```
create table taxi_trips (
    unique_key text, 
    taxi_id text, 
    trip_start_timestamp TIMESTAMP, 
    trip_end_timestamp TIMESTAMP, 
    trip_seconds bigint, 
    trip_miles numeric, 
    pickup_census_tract bigint, 
    dropoff_census_tract bigint, 
    pickup_community_area bigint, 
    dropoff_community_area bigint, 
    fare numeric, 
    tips numeric, 
    tolls numeric, 
    extras numeric, 
    trip_total numeric, 
    payment_type text, 
    company text, 
    pickup_latitude numeric, 
    pickup_longitude numeric, 
    pickup_location text, 
    dropoff_latitude numeric, 
    dropoff_longitude numeric, 
    dropoff_location text
);


COPY taxi_trips(
        unique_key, 
        taxi_id, 
        trip_start_timestamp, 
        trip_end_timestamp, 
        trip_seconds, 
        trip_miles, 
        pickup_census_tract, 
        dropoff_census_tract, 
        pickup_community_area, 
        dropoff_community_area, 
        fare, 
        tips, 
        tolls, 
        extras, 
        trip_total, 
        payment_type, 
        company, 
        pickup_latitude, 
        pickup_longitude, 
        pickup_location, 
        dropoff_latitude, 
        dropoff_longitude, 
        dropoff_location)
    FROM PROGRAM 'awk FNR-1 /tmp/data/taxi*.csv | cat' DELIMITER ',' CSV HEADER;
```

**Общий комментарий**: БД, по которой описано всё выше вессит около 72 ГБ. До этого тестировал так же и на другой БД весом 18 Гб (new_york_taxi_trips.tlc_yellow_trips_2017). В ней первый запрос в PostgreSQL выполнялся чуть больше 3-х минут, в BigQuery все так же менее секунды.
