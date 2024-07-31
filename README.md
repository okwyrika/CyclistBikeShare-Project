## Scenario 

You are a junior data analyst working on the marketing analyst team at Cyclist, a bike-share company in Chicago.The director of the marketing believes the company's future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclist bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members. But first, Cyclist executives must approve your recommendations, so they must be backed up with compelling data insights and professional data visualizations.

## Bussiness Task

My director has assigned me to analyze how annual members and casual riders use Cyclist bikes differently.

## Prepare

The data was made available by Motivate International Inc. Since Cyclist is a Fictional company, the data have a different name. We were asked to download the previous 12 months of Cyclist trip data which I did and stored them in a folder. All personally identifiable information was removed. The data consist of 12 CSV files of a long format,ranging from Jan. 2023 through Dec. 2023. it can be found [here](https://divvy-tripdata.s3.amazonaws.com/index.html). Each file consists of 13 columns:ride_id, rideable_type,started_at,ended_at,start_station_name, start_station_id, end_station_name, end_station_id, start_lat, start_lng, end_lat, end_lng, member_casual. I created a folder for the data I need and named it appropriately.

## Process

I examined the data using excel but the data is large and some columns are not in the right format, so i chose to do the analysis with SQL in MySQL. I imported the data by each month into MYSQL. Prior to importing the data I changed the date column into the right format. I then used the UNION operator to combine all the files into one table, titled "bikeshare_trip".

```{sql}
create table bikeshare_trip as
select ride_id, rideable_type, started_at,ended_at, start_station_name, start_station_id,
end_station_name, end_station_id, start_lat, start_lng, end_lat, end_lng, member_casual
from cyclist_ride.`202301-divvy-tripdata`
union all
select ride_id, rideable_type, started_at,ended_at, start_station_name, start_station_id,
end_station_name, end_station_id, start_lat, start_lng, end_lat, end_lng, member_casual
from cyclist_ride.`202302-divvy-tripdata`
union all
select ride_id, rideable_type, started_at,ended_at, start_station_name, start_station_id,
end_station_name, end_station_id, start_lat, start_lng, end_lat, end_lng, member_casual
from cyclist_ride.`202303-divvy-tripdata`
union all
select ride_id, rideable_type, started_at,ended_at, start_station_name, start_station_id,
end_station_name, end_station_id, start_lat, start_lng, end_lat, end_lng, member_casual
from cyclist_ride.`202304-divvy-tripdata`
```

I repeated this process for all the 12 files.

I then used DISTINCT to start my cleaning process.

```{sql}
select distinct * from bikeshare_trip;
```

I found out that my table have 194298 distinct rows out of 209681 row.
I found no misspelling or spaces in between words. when checking for duplicate entries with the following query, I found out there are 45510 rows that was returned as duplicates.

```{sql}
select ride_id, count(ride_id), rideable_type, started_at, ended_at, 
start_station_name, start_station_id,end_station_name,
end_station_id from bikeshare_trip group by ride_id having count(ride_id) > 1;
```

However most returned rows were not actually duplicates. In order not to remove important entries as duplicates, I used the following query to create a new table "bikeshare_trip_bkp" to get all the distinct entries that I need for my analysis.

```{sql}
create table bikeshare_trip_bkp
as select * from bikeshare_trip 
where 1=2;
```
```{sql}
insert into bikeshare_trip_bkp
select distinct *
from bikeshare_trip;
```

I wrote a query to alter my table by creating  new columns, calculating the length of the ride, the month the ride took place, and the day of the the week the ride took place(1=Sunday, 7=Saturday).

```{sql}
alter table bikeshare_trip_bkp
add trip_duration INT generated always as (ended_at - started_at);
```
```{sql}
select ride_id, rideable_type, started_at, ended_at, dayname(started_at) as day_of_week,
extract(day from started_at) as day_of_month,
extract(month from started_at) as month, start_station_name, start_station_id,
end_station_name, end_station_id, start_lat, start_lng, end_lat, 
end_lng, member_casual, trip_duration
from bikeshare_trip_bkp;
```

## Analyze

### Ride Length

I began my analysis process by finding the average ride length, maximum ride length and minimum ride length for all riders.

```{sql}
select avg(trip_duration) as avg_trip_duration,
max(trip_duration) as max_trip_duration,
min(trip_duration) as min_trip_duration
from bikeshare_trip_bkp;
```

After running the above query, I noticed there are negative ride lengths and some ride lengths lasted for days which signifies a problem. I then wrote these queries to see how many negative rides and rides that lasted for days using the WHERE clause.

```{sql}
select count(*) as neg_trip_duration
from bikeshare_trip_bkp
where trip_duration <0;
```

There are 37133 negative ride lengths.

```{sql}
select count(*) as X_trip_duration
from bikeshare_trip_bkp
where trip_duration >1440;
```
There are 76312 rides that lasted for more than a day.

Going further in this analysis process I will filter out all the negative rides and all rides that lasted for more than a day, as these will give error in my analysis.

My new average, maximum and minimum ride length with filtered data set will now be

```{sql}
select avg(trip_duration) as avg_trip_duration,
max(trip_duration) as max_trip_duration,
min(trip_duration) as min_trip_duration
from bikeshare_trip_bkp
where trip_duration >0 and trip_duration < 1440;
```
Avg. ride length:643.4, Max. ride length:1439, Min. ride length:1

I calculated the average trip duration of members, casual riders

```{sql}
select avg(trip_duration) as avg_trip_duration
from bikeshare_trip_bkp
where trip_duration >0 and
 trip_duration < 1440
 and member_casual ='member';
 ```

 The member's avg. trip duration is 640.4 mins
 
```{sql}
select avg(trip_duration) as avg_trip_duration
from bikeshare_trip_bkp
where trip_duration >0 and
trip_duration < 1440
and member_casual ='casual';
```
 The Casual riders avg. trip duration is 651.7 mins
 
 I went further to calculate the average trip duration of each day of the week for members and casual riders.
 
 ```{sql}
select days, avg(trip_duration) as avg_casual_rides
from bikeshare_trip_bkp
where member_casual ='member'
and trip_duration >0 
and trip_duration <1440
group by days
order by days;
```

```{sql}
select days, avg(trip_duration) as avg_casual_rides
from bikeshare_trip_bkp
where member_casual ='casual'
and trip_duration >0 
and trip_duration <1440
group by days
order by days;
```

Also calculated the average trip duration of members and casual riders by Month.

```{sql}
select extract(month from started_at) as month, avg(trip_duration)
from bikeshare_trip_bkp
where trip_duration >0 and trip_duration < 1440
and member_casual = 'member'
group by month
order by month;
```

```{sql}
select extract(month from started_at) as month, avg(trip_duration)
from bikeshare_trip_bkp
where trip_duration >0 and trip_duration < 1440
and member_casual = 'casual'
group by month
order by month;
```

### Rideable Type

I decided to check if there are differences in the types of vehicles used among member and casual riders.
 
 ```{sql}
select rideable_type, count(rideable_type) as no_of_bike
from bikeshare_trip_bkp
where trip_duration >0 and trip_duration <1440
and member_casual='member'
group by rideable_type;
```

```{sql}
select rideable_type, count(rideable_type) as no_of_bike
from bikeshare_trip_bkp
where trip_duration >0 and trip_duration <1440
and member_casual='casual'
group by rideable_type;
```

### Most Popular Stations

Looking out for the most frequently used stations by member and casual riders, I decided gto first find the most popular start stations for members and casual riders.

```{sql}
select start_station_name, count(*) as rides_taken
from bikeshare_trip_bkp
where member_casual = 'member' 
and trip_duration >0 
and trip_duration <1440
group by start_station_name
order by rides_taken desc;
```

```{sql}
select start_station_name, count(*) as rides_taken
from bikeshare_trip_bkp
where member_casual = 'casual' 
and trip_duration >0 
and trip_duration <1440
group by start_station_name
order by rides_taken desc;
```

Finding the most popular end stations for members and casuals

```{sql]
select end_station_name, count(*) as rides_taken
from bikeshare_trip_bkp
where member_casual = 'member' 
and trip_duration >0 
and trip_duration <1440
group by end_station_name
order by rides_taken desc;
```

```{sql}
select end_station_name, count(*) as rides_taken
from bikeshare_trip_bkp
where member_casual = 'casual' 
and trip_duration >0 
and trip_duration <1440
group by end_station_name
order by rides_taken desc;
```

### Number Of Rides

To know how many rides were taken by members and how many by casuals

```{sql}
select member_casual, count(*) as rides_taken
from bikeshare_trip_bkp
where trip_duration >0 
and trip_duration <1440
group by member_casual;
```

How many rides were taken on each day of the week by member and casual riders?

```[sql}
select dayname(started_at) as days, count(*) as rides_taken
from bikeshare_trip_bkp
where member_casual = 'member' 
and trip_duration >0 
and trip_duration <1440
group by days
order by days;

```{sql}
select dayname(started_at) as days, count(*) as rides_taken
from bikeshare_trip_bkp
where member_casual = 'casual' 
and trip_duration >0 
and trip_duration <1440
group by days
order by days;
```

Finding how many rides was taken by member and casual riders in each month

 ```{sql}
select  Month, count(*) as ride_count
from bikeshare_trip_bkp
where trip_duration >0 and trip_duration<1440
and member_casual = 'member'
group by Month
order by Month;
```

```{sql}
select  Month, count(*) as ride_count
from bikeshare_trip_bkp
where trip_duration >0 and trip_duration<1440
and member_casual = 'casual'
group by Month
order by Month;
```

Checking how many rides were taken at each hour of the day for each day by member and casual riders

```{sql}
select  days, extract(hour from started_at) as hours,
count(*) as rides_taken
from bikeshare_trip_bkp
where trip_duration >0 
and trip_duration <1440
and member_casual = 'member'
group by days, hours
order by days, hours desc;
```

```{sql}
select  days, extract(hour from started_at) as hours,
count(*) as rides_taken
from bikeshare_trip_bkp
where trip_duration >0 
and trip_duration <1440
and member_casual = 'casual'
group by days, hours
order by days, hours desc;
```

## Findings
-
