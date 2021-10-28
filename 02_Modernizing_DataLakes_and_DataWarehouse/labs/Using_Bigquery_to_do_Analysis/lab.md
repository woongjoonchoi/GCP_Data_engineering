# Using_Bigquery_to_do_Analysis  explanation

## 1.sql
```sql
SELECT
  MIN(start_station_name) AS start_station_name,
  MIN(end_station_name) AS end_station_name,
  APPROX_QUANTILES(tripduration, 10)[OFFSET (5)] AS typical_duration,
  COUNT(tripduration) AS num_trips
FROM
  `bigquery-public-data.new_york_citibike.citibike_trips`
WHERE
  start_station_id != end_station_id
GROUP BY
  start_station_id,
  end_station_id
ORDER BY
  num_trips DESC
LIMIT
  10


```

### APPROX_QUANTAILE
```sql
APPROX_QUANTILES(
  [DISTINCT]
  expression, number
  [{IGNORE|RESPECT} NULLS]
)


SELECT APPROX_QUANTILES(x, 2) AS approx_quantiles
FROM UNNEST([1, 1, 1, 4, 5, 6, 7, 8, 9, 10]) AS x;

+------------------+
| approx_quantiles |
+------------------+
| [1, 5, 10]       |
+------------------+


SELECT APPROX_QUANTILES(x, 100)[OFFSET(90)] AS percentile_90
FROM UNNEST([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]) AS x;

+---------------+
| percentile_90 |
+---------------+
| 9             |
+---------------+
SELECT APPROX_QUANTILES(DISTINCT x, 2) AS approx_quantiles
FROM UNNEST([1, 1, 1, 4, 5, 6, 7, 8, 9, 10]) AS x;

+------------------+
| approx_quantiles |
+------------------+
| [1, 6, 10]       |
+------------------+

```

Description:It returns array length number +1 ,  Elements are quantiles in expression separate by number sections 
https://cloud.google.com/bigquery/docs/reference/standard-sql/approximate_aggregate_functions#approx_quantiles

### ARRAY OFFSET
```sql

 [ OFFSET  (number) ]
 
 WITH sequences AS
  (SELECT [0, 1, 1, 2, 3, 5] AS some_numbers
   UNION ALL SELECT [2, 4, 8, 16, 32] AS some_numbers
   UNION ALL SELECT [5, 10] AS some_numbers)
SELECT some_numbers,
       some_numbers[OFFSET(1)] AS offset_1,
       some_numbers[ORDINAL(1)] AS ordinal_1
FROM sequences;

+--------------------+----------+-----------+
| some_numbers       | offset_1 | ordinal_1 |
+--------------------+----------+-----------+
| [0, 1, 1, 2, 3, 5] | 1        | 0         |
| [2, 4, 8, 16, 32]  | 4        | 2         |
| [5, 10]            | 10       | 5         |
+--------------------+----------+-----------+
```

Description: It returns position number element in ARRAY

https://cloud.google.com/bigquery/docs/reference/standard-sql/arrays

### Query Explanation
This query returns most(Limit 10 DESC num_trips)  common duration( indexing 5 in 10 sections quantile)


## 2.sql
```sql
WITH
  trip_distance AS (
SELECT
  bikeid,
  ST_Distance(ST_GeogPoint(s.longitude,
      s.latitude),
    ST_GeogPoint(e.longitude,
      e.latitude)) AS distance
FROM
  `bigquery-public-data.new_york_citibike.citibike_trips`,
  `bigquery-public-data.new_york_citibike.citibike_stations` as s,
  `bigquery-public-data.new_york_citibike.citibike_stations` as e
WHERE
  start_station_id = s.station_id
  AND end_station_id = e.station_id )
SELECT
  bikeid,
  SUM(distance)/1000 AS total_distance
FROM
  trip_distance
GROUP BY
  bikeid
ORDER BY
  total_distance DESC
LIMIT
  5

```
### Query Explanation

Top 5 (LIMIT 5) biketrip distance

## 3.sql

```sql

SELECT
  wx.date,
  wx.value/10.0 AS prcp
FROM
  `bigquery-public-data.ghcn_d.ghcnd_2015` AS wx
WHERE
  id = 'USW00094728'
  AND qflag IS NULL
  AND element = 'PRCP'
ORDER BY
  wx.date

```
### Query Explanation

All rainy day NEW YORK CNTRL PK TWR station( id = 'USW00094728')

## 4.sql

```sql
WITH bicycle_rentals AS (
  SELECT
    COUNT(starttime) as num_trips,
    EXTRACT(DATE from starttime) as trip_date
  FROM `bigquery-public-data.new_york_citibike.citibike_trips`
  GROUP BY trip_date
),
rainy_days AS
(
SELECT
  date,
  (MAX(prcp) > 5) AS rainy
FROM (
  SELECT
    wx.date AS date,
    IF (wx.element = 'PRCP', wx.value/10, NULL) AS prcp
  FROM
    `bigquery-public-data.ghcn_d.ghcnd_2015` AS wx
  WHERE
    wx.id = 'USW00094728'
)
GROUP BY
  date
)
SELECT
  ROUND(AVG(bk.num_trips)) AS num_trips,
  wx.rainy
FROM bicycle_rentals AS bk
JOIN rainy_days AS wx
ON wx.date = bk.trip_date
GROUP BY wx.rainy

```

### EXTRACT

```sql
EXTRACT(part FROM date_expression)
```
Description : Return the specificed Date
https://cloud.google.com/bigquery/docs/reference/standard-sql/date_functions#extract

### ROUND


```sql

ROUND(X [, N])
```
Description :  Return nearst integer to X , If N i present , Rounds X to N decimal places after the decimal point
https://cloud.google.com/bigquery/docs/reference/standard-sql/mathematical_functions#round


### Query Explanation

Join rainy day and biketrip(FROM bicycle_rentals AS bk
JOIN rainy_days AS wx) . Then , return average trips in rainy day(ON wx.date = bk.trip_date)


### Query Result

![image](https://user-images.githubusercontent.com/50165842/139187337-b50cc8aa-78d1-4411-95da-6bda9b12df27.png)

New Yorkers ride the bicycle 47% fewer times when it rains.


