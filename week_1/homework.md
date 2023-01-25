## Question 1.
--iidfile string
## Question 2.
Command output:

    Package    Version
    ---------- -------
    pip        22.0.4
    setuptools 58.1.0
    wheel      0.38.4

## Question 3. Count trips in 2019-01-15
Query:
```sql
select count(*)
	from public.green_taxi_trips
where date(lpep_pickup_datetime)='2019-01-15'
and date(lpep_dropoff_datetime)='2019-01-15'
```
Output:

    +-------+
    | count |
    +-------|
    | 20530 |
    +-------+

## Question 4. Largest trip distance
Query:
```sql
select lpep_pickup_datetime::date, trip_distance
	from public.green_taxi_trips
order by trip_distance desc
limit 1
```
Output:

    +----------------------+------------------+
    | lpep_pickup_datetime | trip_distance    |
    |----------------------+------------------|
    | 2019-01-15           | 117.99           |
    +----------------------+------------------+

## Question 5. The number of passengers, start trip in 2019-01-01
Query:
```sql
select distinct passenger_count, count(*) over(partition by passenger_count)
	from public.green_taxi_trips
where date(lpep_pickup_datetime)='2019-01-01'
and passenger_count between 2 and 3
```
Output:

    +-----------------+-------+
    | passenger_count | count |
    +-----------------+-------|
    | 2               | 1282  |
    | 3               | 254   |
    +-----------------+-------+

## Question 6. Largest tip with out Astoria
Query:
```sql
select "Zone" as zone_dropoff_loc
  from public.green_taxi_trips join public.zone_trips
on "DOLocationID"="LocationID"
	where "PULocationID" = (select "LocationID"
	from public.zone_trips where "Zone"='Astoria')
order by tip_amount desc
limit 1
```
Output:

    +-------------------------------+
    | zone_dropoff_loc              |
    |-------------------------------|
    | Long Island City/Queens Plaza |
    +-------------------------------+
  
