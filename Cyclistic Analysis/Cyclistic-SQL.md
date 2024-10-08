## Cyclistic Data Analysis

I'm a professional writer. As such, I have a more verbose [walkthrough of my thinking](https://docs.google.com/document/d/1TJh7n6wp2OoD5BLZqpVwCtvDQYmGWW2nXmEgKdBO1nA/edit?usp=sharing)
during this analysis project for those interested.

- To begin this data analysis capstone, I grabbed the Q4 2019 data from [this data set](https://divvy-tripdata.s3.amazonaws.com/index.html).

### Questions
Where might this company need to be sure to allocate its support and maintenance resources?

Are there opportunities to convert one-off customers (i.e., subscribers) in the data? Are there differences between the highest ride origin stations when comparing subscribers vs. customers?

Finally, to optimize marketing efforts, what are the days of highest bike utilization?

### Cleaning

```sql
# Checked for duplicate entries. Here is the query.
SELECT distinct trip_id
FROM `your-project.cyclistic_2019.rides_q1_2019`
GROUP BY trip_id

# dropped one of the columns from the table.
ALTER TABLE `your-project.cyclistic_2019.rides_q1_2019`
DROP COLUMN bikeid;

# Removed null values for the start time and end time. Did not find null values.
SELECT * FROM `promising-rock-425020-h8.cyclistic_2019.rides_q1_2019` 
WHERE start_time IS NULL or end_time IS NULL;

DELETE FROM `promising-rock-425020-h8.cyclistic_2019.rides_q1_2019`
WHERE start_time IS NULL
  OR end_time IS NULL;
```

### Manipulation

```sql
# Time to calculate the days of the week for each trip.
ALTER TABLE `promising-rock-425020-h8.cyclistic_2019.rides_q1_2019`
 ADD COLUMN day_of_week INTEGER;

# Extracted the day of week value from the start_time time stamp and populated the day_of_week column.
UPDATE `promising-rock-425020-h8.cyclistic_2019.rides_q1_2019`
SET day_of_week = EXTRACT(dayofweek from start_time)
WHERE start_time < end_time;
```

### Analysis

```sql
# Stations have the highest usage?
SELECT from_station_name,
 count(trip_id) as trips
FROM promising-rock-425020-h8.cyclistic_2019.rides_q1_2019
GROUP BY from_station_name
ORDER BY trips DESC
LIMIT 5;

# Stations with highest usage among members/subscribers.
SELECT from_station_name,
	count(trip_id) as trips
FROM promising-rock-425020-h8.cyclistic_2019.rides_q1_2019
WHERE usertype = 'Subscriber'
GROUP BY from_station_name
ORDER BY trips DESC
LIMIT 5;
```

- Remember: column names are case-sensitive! Was stuck for a while when query didn’t return any results.

```sql
# Most popular days of the week.
SELECT day_of_week,
 count(trip_id) as trips
FROM promising-rock-425020-h8.cyclistic_2019.rides_q1_2019
GROUP BY day_of_week
ORDER BY trips DESC;

# Converted days of week from numeric values to the actual days, or at least the abbreviations. 
# Finally figured out a way to do it.
SELECT
 day_of_week as numeric_day,
 count(trip_id) AS trips,
 CASE
   WHEN day_of_week = 1 THEN 'Sun'
   WHEN day_of_week = 2 THEN 'Mon'
   WHEN day_of_week = 3 THEN 'Tue'
   WHEN day_of_week = 4 THEN 'Wed'
   WHEN day_of_week = 5 THEN 'Thu'
   WHEN day_of_week = 6 THEN 'Fri'
   WHEN day_of_week = 7 THEN 'Sat'
   ELSE 'NA'
 END AS weekday
FROM promising-rock-425020-h8.cyclistic_2019.rides_q1_2019
GROUP BY day_of_week
ORDER BY day_of_week;
```
