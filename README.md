# Chicago Cyclist Bike Share Analysis using Microsoft SQL Server and Tabeau Public

Scenario:
In 2016, Cyclistic launched a successful bike-share offering. The company’s future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members.

This case study follows the six steps in the data analysis Process:
- Ask
- Prepare
- Process
- Analyze
- Share
- Act

# 1. ASK

- Define the stakeholders: Director of Marketing (Lily Moreno) and the Cylcistic Executive Team
- 1. How do casual users and annual subcribed members use Cyclistic Bikes differently?
- 2. How can we design new marketing strategies to help convert casual members into annual members?

# 2. Prepare
- The data source 12 months of dat (Dec 2020 - Nov. 2021) [Data Source Link](https://divvy-tripdata.s3.amazonaws.com/index.html)
- I also used the ROCC approach for this data to determine credibility
- Reliability:  The dataset includes accurate ride data from Divvy. Divvy is a program of Chicago Department of Transportation (CDOT), and they own the city's bikes and stations. Data integrity was ensured through periodically valaditing the input source of the data.
- Original: The data is from Motivate International Inc which is a Chicago Bicycle Sharing Company
- Current: Data is up to date Nov 2021
- Cited: The data falls under [this license agreement](https://ride.divvybikes.com/data-license-agreement) 

# 3. Process
- I used Microsoft SQL Server to clean and transform the data
- I began by combining all 12 months of data into one table:
```
------ Combining tables into one ------
SELECT * INTO tot_data

FROM

(
SELECT * from bikeshare.dbo.dec_2020$
UNION all
SELECT * from bikeshare.dbo.jan_2021$
UNION ALL
SELECT * from bikeshare.dbo.feb_2021$
UNION all
SELECT * from bikeshare.dbo.mar_2021$
union all
SELECT * from bikeshare.dbo.apr_2021$
UNION all
SELECT * from bikeshare.dbo.may_2021$
UNION ALL
SELECT * from bikeshare.dbo.june_2021$
UNION all 
SELECT * from bikeshare.dbo.july_2021$
UNION all
SELECT * from bikeshare.dbo.aug_2021$
UNION all
SELECT * from bikeshare.dbo.sep_2021$
UNION all 
SELECT * from bikeshare.dbo.oct_2021$
UNION all
SELECT * from bikeshare.dbo.nov_2021$)

```
- I then added a column to calcuate ride duration 

```
-----Adding a new column to calcuate each ride duration--------

ALTER TABLE tot_data
ADD duration int

UPDATE tot_data
SET duration = DATEDIFF(MINUTE, started_at, ended_at)
```
- I extracted the month and year, and added them as new columns
```
ALTER TABLE tot_data
ADD day_of_week nvarchar(50),
month_m nvarchar(50),
year_y nvarchar(50)

UPDATE tot_data
SET day_of_week = DATENAME(WEEKDAY, started_at),
month_m = DATENAME(MONTH, started_at),
year_y = year(started_at)

--Creating month integer column
ALTER TABLE tot_data       
ADD month_int int

UPDATE tot_data          
SET month_int = DATEPART(MONTH, started_at)
```
- Data cleaning: I trimed the station name to ensure there is no extra space. I also filtered out rows with (LBS-WH-TEST) in the start and end station names

```
ALTER TABLE tot_data
ADD start_station_name_cleaned nvarchar(255)



Update tot_data
Set start_station_name_cleaned = TRIM(REPLACE
(REPLACE
(start_station_name, '(*)',''), '(TEMP)','')) 
FROM tot_data
WHERE start_station_name NOT LIKE '%(LBS-WH-TEST)%' 

ALTER TABLE tot_data
ADD end_station_name_cleaned nvarchar(255)

Update tot_data
Set end_station_name_cleaned = TRIM(REPLACE
(REPLACE
(start_station_name, '(*)',''), '(TEMP)','')) 
FROM tot_data
WHERE end_station_name NOT LIKE '%(LBS-WH-TEST)%'
```

- Deleted null values

```
---- Deleted rows where (NULL values), (ride length = 0), (ride length < 0), (ride_length > 1 day (1440 mins)) for accurate analysis -----

DELETE FROM tot_data
Where ride_id IS NULL OR
start_station_name IS NULL OR
end_station_name IS NULL OR
start_station_id IS NULL OR
end_station_id IS NULL OR
duration IS NULL OR
duration = 0 OR
duration < 0 OR
duration > 1440 
```
- Checked for any duplicates and renamed column 'member_casual' to 'user_type'
```
-- Checking for any duplicates

Select Count(DISTINCT(ride_id)) AS uniq,
Count(ride_id) AS total
From tot_data_cleaned

--Rename member_casual to user_type
EXEC sp_RENAME 'tot_data.member_casual', 'user_type', 'COLUMN'
```
# 4. Analyze
- Compared casual riders vs members, and grouped by the day of the week

```
----- Members vs Casual riders grouped by day of the week------

Create View users_per_day AS
Select 
Count(case when user_type = 'member' then 1 else NULL END) AS num_of_members,
Count(case when user_type = 'casual' then 1 else NULL END) AS num_of_casual,
Count(*) AS num_of_users,
day_of_week
FROM tot_data
GROUP BY day_of_week
```
- Calculated the average ride length for each user type

```
Create View avg_ride_length AS
SELECT user_type, AVG(duration) AS avg_ride_length, day_of_week 
From tot_data
Group BY user_type, day_of_week

```

- User traffic every month since startup
```
-- Calculating User Traffic Every Month Since Startup

Select month_int AS Month_Num,
month_m AS Month_Name, 
year_y AS Year_Y,
Count(case when user_type = 'member' then 1 else NULL END) AS num_of_member,
Count(case when user_type = 'casual' then 1 else NULL END) AS num_of_casual,
Count(user_type) AS total_num_of_users
From tot_data
Group BY year_y, month_int, month_m
ORDER BY year_y, month_int, month_m

```
- Compared casual riders vs members by the month
```
--casual vs member by month
Select month_m, 
Count(case when user_type = 'member' then 1 else NULL END) AS num_of_member,
Count(case when user_type = 'casual' then 1 else NULL END) AS num_of_casual
From tot_data
Group BY month_m
```
- Top 5 start stations for casual riders
```
Select top 5 start_station_name, 
Count(case when user_type = 'casual' then 1 else NULL END) AS num_of_casual
From tot_data
Group BY start_station_name
Order by num_of_casual desc
```
