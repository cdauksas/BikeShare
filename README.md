# Chicago Cyclist Bike Share Analysis

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
- I exracted the month and year, and added them as new columns
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
