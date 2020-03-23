# Homework

## Instructions

Clone this repo and make a PR with your answers to the questions below, and tag your reviewer in the PR.

Keep good notes of the queries you run.

## Part 1: Missing Station Data

As we've discussed in class, some of the station_id columns are NULL because of missing data. We also have inconsistent names for some of the station IDs.

Your database contains a `stations` schema:

```sql
CREATE TABLE stations
(
  id INT NOT NULL PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);
```

1. Write a query that fills out this table. Try your best to pick the correct station name for each ID. You may have to make some manual choices or editing based on the inconsistencies we've found. Do try to pick the correct name for each station ID based on how popular it is in the trip data.

Hint: your query will look something like

```sql
INSERT INTO stations (SELECT ... FROM tripe);
```

``` sql
INSERT INTO stations
(SELECT  from_station_id as id, max(distinct from_station_name)  as name
from trips

WHERE from_station_id IS NOT NULL group by from_station_id HAVING COUNT(distinct from_station_name) = 1 OR COUNT(distinct from_station_name) > 1

union

SELECT  to_station_id as id, max(distinct to_station_name)  as name
from trips

WHERE to_station_id IS NOT NULL group by to_station_id HAVING COUNT(distinct to_station_name) = 1 OR COUNT(distinct to_station_name) > 1

ORDER BY id ASC);
// this query assumes that the frequency of the correct station_name would be more than the frequency of the incorrect one for a particular station_id ==> hence the use of the max aggregate function.

--- [Table: stations] is a mapping of station id and station name
--- There is a possibility that a 'to_station_name' might not be a 'from_station_name'
But even in that case ==> 'to_station_id' and 'to_station_name' has to be inserted in [Table: stations]
--- Hence the reason why an Union has been done between from_station_id, from_station_name AND
to_station_id, to_station_name. Union removes duplicates. 

```

Hint 2: You don't have to do it all in one query

2. Should we add any indexes to the stations table, why or why not?

- An index could be used here to speed up the searching but it would be more effective/useful in a larger database \*.

3. Fill in the missing data in the `trips` table based on the work you did above

```
UPDATE trips
SET from_station_id = stations.id
FROM stations WHERE trips.from_station_name = stations.name;

UPDATE trips
SET to_station_id = stations.id
FROM stations WHERE trips.to_station_name = stations.name;

```

## Part 2: Missing Date Data

You may have noticed that we have the dates as strings. This is because the dataset we have uses an inconsistent format for dates 😔😔😔

Note that the `original_filename` column is broken down into quarters, knowing this is helpful here!

1. What's the inconsistency in date formats? You can assume that each quarter's trips are numbered sequentially, starting with the first day of the first month of that quarter.

2. Take a look at Postgres's [date functions](https://www.postgresql.org/docs/12/functions-datetime.html), and fill in the missing date data using proper timestamps. You may have to write several queries to do this.

Hint: your queries will look something like

```sql
UPDATE trips
SET start_time = ..., end_time = ...
WHERE ...;
```

3. Other than the index in class, would we benefit from any other indexes on this table? Why or why not?

## Part 3: Data-driven insights

Using the table you made in part 1 and the dates you added in part 2, let's answer some questions about the bike share data

1. Build a mini-report that does a breakdown of number of trips by month
2. Build a mini-report that does a breakdown of number trips by time of day of their start and end times
3. What are the most popular stations to bike to in the summer?
4. What are the most popular stations to bike from in the winter?
5. Come up with a question that's interesting to you about this data that hasn't been asked and answer it.
