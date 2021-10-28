# SQL explanation

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

Query Explanation:  

