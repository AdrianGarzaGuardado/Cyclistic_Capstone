# Cyclistic_Capstone
Cyclistic Bike Share Case Study

## Introduction

Cyclistic bike-share analysis is a case study from the Google Data Analytics Professional Certificate, in this case you work for a fictional company, Cyclistic. In order to answer the business questions, you will follow the steps of the data analysis process: Ask, Prepare, Process, Analyze, Share, and Act.

You are a junior data analyst working on the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members. But first, Cyclistic executives must approve your recommendations, so they must be backed up with compelling data insights and professional data visualizations.

## Background

In 2016, Cyclistic launched a successful bike-share offering. Since then, the program has grown to a fleet of 5,824 bicycles that are geotracked and locked into a network of 692 stations across Chicago. The bikes can be unlocked from one station and returned to any other station in the system anytime.

Until now, Cyclistic’s marketing strategy relied on building general awareness and appealing to broad consumer segments. One approach that helped make these things possible was the flexibility of its pricing plans: single-ride passes, full-day passes, and annual memberships. Customers who purchase single-ride or full-day passes are referred to as casual riders. Customers who purchase annual memberships are Cyclistic members.

Cyclistic’s finance analysts have concluded that annual members are much more profitable than casual riders. Although the pricing flexibility helps Cyclistic attract more customers, the company´s Marketing Director believes that maximizing the number of annual members will be key to future growth. Rather than creating a marketing campaign that targets all-new customers, there is a solid opportunity to convert casual riders into members. The Director notes that casual riders are already aware of the Cyclistic program and have chosen Cyclistic for their mobility needs.

## 1. ASK
*Deliverable: a clear statement of the business task*

Design marketing strategies aimed at converting casual riders into annual members, In order to do that, the team needs to better understand:

1.	How do annual members and casual riders use Cyclistic bikes differently?
2.	Why would casual riders buy Cyclistic annual memberships?
3.	How can Cyclistic use digital media to influence casual riders to become members?

I have been assigned by the Marketing Director to answer the first question, how do annual members and casual riders use Cyclistic bikes differently?

## 2. PREPARE
*Deliverable: a description of all data sources used*

Data source: link
Scope: 12 files, one per month of the year 2024
License: data has been made available by Motivate International Inc. under this license

Each data table consists of 13 variables with the following data type and description:

No.	Variable	Type	Description
1	ride_id	STRING	ID assigned to each ride
2	rideable_type	STRING	Type of bike category
3	started_at	TIMESTAMP	Date and time at the start of the trip
4	ended_at	TIMESTAMP	Date and time at the end of the trip
5	start_station_name	STRING	Station name at start of the trip
6	start_station_id	STRING	ID of start station name
7	end_station_name	STRING	Station name at end of the trip
8	end_station_id	STRING	ID of end station name
9	start_lat	FLOAT	Latitude coordinate of start station
10	start_lng	FLOAT	Longitude coordinate of start station
11	end_lat	FLOAT	Latitude coordinate of end station
12	end_lng	FLOAT	Longitude coordinate of end station
13	member_casual	STRING	Type of membership category

## 3. PROCESS
*Deliverables: Check the data for errors, choose your tools, transform the data so you can work with it effectively and document the cleaning process*

### Combining data

Since the data set is too large (6,355,568 rows all files combined) for programs such as Excel or Sheets, I chose BigQuery’s SQL platform to work effectively with the datasets. 

After uploading each of the last 12 month datasets (from January to December 2024), I combined them into one using the following query:
```sql
CREATE TABLE cyclistic.cyclistic_2021_all AS (
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202401` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202402` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202403` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202404` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202405_1` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202405_2` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202406_1` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202406_2` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202406_3` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202407_1` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202407_2` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202408_1` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202408_2` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202409_1` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202409_2` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202410_1` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202410_2` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202411` UNION ALL
  SELECT * FROM `swift-castle-448513-q4.cyclistic.202412` 
);
```
### Check the data for errors.

#### 1) Check that the number of strings for ride_id is uniform
```sql
SELECT LENGTH(ride_id) AS ride_id_len
FROM `swift-castle-448513-q4.cyclistic.cyclistic_2024_all`
GROUP BY LENGTH(ride_id)
```
Row	ride_id_len	f0_
1	16	6355568

NOTES: All ride_id strings are 16 characters long

#### 2) Check the amount of rideable_type categories
```sql
SELECT rideable_type, COUNT(*)
FROM `swift-castle-448513-q4.cyclistic.cyclistic_2024_all`
GROUP BY rideable_type;
```
Row	rideable_type	f0_
1	electric_bike	3213028
2	electric_scooter	144337
3	classic_bike	2998203

NOTES: there are 3 categories of ‘rideable_type’: electric_bike, electric_scooter and classic_bike. electric_scooter is not in the scope of the analysis so these need to be removed in the cleaning process.

#### 3) Check for MIN and MAX values of ride length to understand data.
```sql
SELECT
  MAX (ride_length) AS max_lenght,
  MIN (ride_length) AS min_lenght
FROM `swift-castle-448513-q4.cyclistic.cyclistic_2024_all`
```
Row	max_lenght	min_lenght
1	1560	-2748

NOTES: determined that there are negative ride length values, these are errors that must be eliminated. Maximum value is 26 hours which belong to a accepted range.

#### 4) Identify records where ride duration time is less than a minute or negative
```sql
SELECT ride_id, started_at, ended_at, ride_length
FROM `swift-castle-448513-q4.cyclistic.cyclistic_2024_all`
WHERE TIMESTAMP_DIFF(ended_at, started_at, MINUTE) <= 1
ORDER BY ride_length;
```
NOTES: a total of 190,100 records identified with less than a minute or negative ride length, these need to be removed in the cleaning process as don’t add value to the analysis.

#### 5) Check the amount of member_casual categories
```sql
SELECT member_casual, COUNT(*)
FROM `swift-castle-448513-q4.cyclistic.cyclistic_2024_all`
GROUP BY member_casual;
```
Row	member_casual	f0_
1	casual	2343957
2	member	4011611

NOTES: Confirmed that the only 2 values in this field are ‘member’ and ‘casual’.

#### 6) Check total number of null values from each column
```sql
SELECT 
  COUNT (*) - COUNT (ride_id) AS ride_id_nulls,
  COUNT (*) - COUNT (rideable_type) AS rideable_type_nulls,
  COUNT (*) - COUNT (started_at) AS started_at_nulls,
  COUNT (*) - COUNT (ended_at) AS ended_at_nulls,
  COUNT (*) - COUNT (start_station_name) AS start_station_name_nulls,
  COUNT (*) - COUNT (start_station_id) AS start_station_id_nulls,
  COUNT (*) - COUNT (end_station_name) AS end_station_name_nulls,
  COUNT (*) - COUNT (end_station_id) AS end_station_id_nulls,
  COUNT (*) - COUNT (start_lat) AS start_lat_nulls,
  COUNT (*) - COUNT (start_lng) AS start_lng_nulls,
  COUNT (*) - COUNT (end_lat) AS end_lat_nulls,
  COUNT (*) - COUNT (end_lng) AS end_lng_nulls,
  COUNT (*) - COUNT (member_casual) AS member_casual_nulls
FROM `swift-castle-448513-q4.cyclistic.cyclistic_2024_all`;
```
ride_id_nulls 	0
rideable_type_nulls 	0
started_at_nulls 	0
ended_at_nulls 	0
start_station_name_nulls 	1140020
start_station_id_nulls 	1140020
end_station_name_nulls 	1168579
end_station_id_nulls 	1168579
start_lat_nulls 	0
start_lng_nulls 	0
end_lat_nulls 	7867
end_lng_nulls 	7867
member_casual_nulls 	0

NOTES: Confirmed that key columns such as ride_id, rideable_type, started_at, ended_at and member_casual don´t have null values, I will not delete nulls since these records contain valuable information. 

### Data Cleaning

1) Added 2 more columns to the table: ride_length for duration of the trip and day_of_week.
2) Removed trips with duration less than a minute.
3) Remove records with rideable_type equal to “electric_scooter”
```sql
CREATE TABLE IF NOT EXISTS cyclistic.cyclistic_2024_final AS (
SELECT
  ride_id,
  rideable_type,
  started_at,
  ended_at,
  start_station_name,
  start_station_id,
  end_station_name,
  end_station_id,
  start_lat,
  start_lng,
  end_lat,
  end_lng,
  member_casual,
  TIMESTAMP_DIFF (ended_at, started_at, MINUTE) AS ride_length,
  EXTRACT (DAYOFWEEK FROM started_at) AS day_of_week,
FROM `swift-castle-448513-q4.cyclistic.cyclistic_2024_all`
WHERE
  TIMESTAMP_DIFF (ended_at, started_at, MINUTE) > 1 AND
  rideable_type = "electric_bike" OR
  rideable_type = " classic_bike" 
);
```

## 4. ANALYZE
*Deliverable: A summary of your analysis*

The analysis will focus on answering the question: How do annual members and casual riders use Cyclitic bikes differently?

Overview of 2024 data by member type:

![Picture8](https://github.com/user-attachments/assets/1773dc90-37e0-4046-abd0-2f3f4b761533)

* Casual riders represent only 36% of the total amount of rides, but contributed to 53% of total ride length
* Casual rides have an average ride length of 26.7 min/ride while members have an average of 13.1 min/ride
* Casual riders take less number of trips but with almost two times longer ride lengths than members.

Overview of 2024 data by rideable type:

![Picture7](https://github.com/user-attachments/assets/bb4912ed-b80e-45c0-b73c-df675a753f3b)

* Electric bikes represented 36.6% of the total number of rides
* Cyclitic members have a bigger share of electric bikes with 45% compared to casual riders with 30%.
* Casual riders show a higher preference for classic bikes.

Ride behavior by day of the week:

![Picture6](https://github.com/user-attachments/assets/57d5c469-218a-4903-a5a5-87ba468372b5)

* Members use the riding service more often during weekdays, the hypothesis is that members use the service primarily to commute to work.
* Casual riders have more riding activity during the weekends, the hypothesis is that their primary purpose is for leisure and tourism.

Ride behavior by hour of the day:

For the following analysis I created a calculated field in Tableau to extract the hour part of the datetime measure to identify usage volume by hour of the day.

STR(DATEPART('hour',[Started At]))

 ![Picture5](https://github.com/user-attachments/assets/4d4c67e8-c7e0-46cc-810b-a98999cbc37c)

* Cyclistic members have the most volume of rides during work commuting hours, between 6:00am to 9:00am and 3:00pm to 7:00pm
* Casual riders have the most volume of rides on weekends during daylight hours from 10am to 6pm
* Casual riders, when excluding weekends, show a significant volume of rides from 3pm to 7pm, these volumes represent an opportunity on capturing membership rides.

Ride behavior by month:

![Picture4](https://github.com/user-attachments/assets/f0ddfd7e-c75e-4ba1-8355-aca6e03264e5)

* The months from May to October contribute to 78% of total year rides.
* The graph above show that Cyclitic service usage can be correlated to weather conditions, having the hottest months of the year a significant increase of volume.

Rideable type of bike trend:

![Picture3](https://github.com/user-attachments/assets/088df2ff-c976-4e60-9163-9e67f3cdf28a)

* There is an increasing trend in usage of electric bikes, seeing an increase from 30% of total rides starting in January to almost 50% of total rides at the end of the year.
* Both casual and members have an increasing trend on the number of electric bikes per month.

Identification of most common stations:

The following map displays the most 15 common stations by total count of rides in 2024

 ![Picture2](https://github.com/user-attachments/assets/09e9da57-17b3-4343-90b4-0cb7ab74b1f4)

## 5. SHARE
*Deliverable: Supporting visualizations and key findings*

Cyclistc analysis tableau dashboard link

 ![Picture1](https://github.com/user-attachments/assets/2381d14c-ac16-47a7-a82b-d7549c0a9312)

Differences:
* Casual riders average ride length is twice longer than members (26.7 vs 13.1 min/ride).
* Casual riders take less trips than members but contribute to more riding minutes (36% of total amount of trips represented 53% of total ride length).
* Casual riders have a lower preference of electric bikes (30% vs 45% of share).
* Casual riders take significatively more trips on weekends while members ride more often during weekdays.
* Casual riders ride often during daylight hours for leisure while members ride more on work commuting hours.

Similarities:
* Both take more trips in summer or in correlation to warm weather conditions, the months from May to October contribute to 78% of total year rides.
* Both have more preference for regular bikes than electric.
* Both have the same top start and end stations.

## 6. ACT
*Deliverable: Your top three recommendations based on your analysis*

Following are my recommendations for designing marketing strategies to convert casual riders to Cyclistic members:

1.	Design shorter membership plans valid exclusively from May to October. These months represent 80% of casual riders yearly demand.
2.	Design weekend exclusive membership plans since casual riders are more active on Saturdays and Sundays.	
3.	Provide discounts or incentives for long length trips since casuals ride for longer time.

