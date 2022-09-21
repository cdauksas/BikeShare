# Chicago Cyclist Bike Share Analysis

** Date: September 20, 2021  **

Scenario:
In 2016, Cyclistic launched a successful bike-share offering. The company’s future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members.

1. How do casual users and annual subcribed members use Cyclistic Bikes differently?
2. How can we design new marketing strategies to help convert casual members into annual members?
- Begin by combining all 12 months together into one table:
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
-
