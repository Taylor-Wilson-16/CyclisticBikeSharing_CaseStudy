# DivvyBikeSharing_CaseStudy2
Google Data Analytics Capstone Project


### DOWNLOAD AND CLEAN DATA

# Create table and rename new combined data
CREATE TABLE `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023` AS (

# Combine all datasets vertically using UNION ALL
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202301_DivvyTripData` 
UNION ALL
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202302_DivvyTripData` 
UNION ALL
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202303_DivvyTripData`
UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202304_DivvyTripData`

UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202305_DivvyTripData1` # Large datasetse split into multiple smaller datasets to import
UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202305_DivvyTripData2`
UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202305_DivvyTripData3`

UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202306_DivvyTripData1`
UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202306_DivvyTripData2`
UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202306_DivvyTripData3`

UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202307_DivvyTripData1`
UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202307_DivvyTripData2`
UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202307_DivvyTripData3`

UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202308_Divvy_TripData1`
UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202308_DivvyTripData2`
UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202308_DivvyTripData3`

UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202309_DivvyTripData1`
UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202309_DivvyTripData2`
UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202309_DivvyTripData3`

UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202310_DivvyTripData1`
UNION ALL 
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202310_DivvyTripData2`

UNION ALL
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202311_DivvyTripData`
UNION ALL
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.202312_DivvyTripData`
)

# Create new table with cleaned data and new columns
CREATE TABLE `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned` AS (
SELECT 
  DISTINCT ride_id,
  rideable_type,
  started_at,
  ended_at,
  DATE_DIFF(ended_at, started_at, minute) AS ride_length, # Create new ride_length column
  FORMAT_DATE('%A', started_at) AS day_of_week, # Create new day_of_week column
  FORMAT_DATE('%B', started_at) AS month, # Create new month column
  TIME(started_at) AS time, # Create new time column
  member_casual,
  start_station_name,
  end_station_name
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023`
WHERE DATE_DIFF(ended_at, started_at, minute) > 0
ORDER BY ride_length DESC # Sort by ride_length 
)
# Check that data types are correct
# Check that ended_at - started_at = ride_length

# Limitation: There are 6.417 rides that were longer than 24 hours.
# Remove rides > 24 hours 
CREATE OR REPLACE TABLE `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned` AS (
SELECT *
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned`
WHERE ride_length < 1440
)

##ANALYZE DATA
# Count number of member rides and casual rides
SELECT
  COUNTIF(member_casual = 'member') AS member_rides,
  COUNTIF(member_casual = 'casual') AS casual_rides
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned`

# Average, maximum, and minimum  ride length
SELECT 
  member_casual,
  AVG(DATE_DIFF(ended_at, started_at, minute)) AS avg_ride_length,
  MAX(DATE_DIFF(ended_at, started_at, minute)) AS max_ride_length,
  MIN(DATE_DIFF(ended_at, started_at, minute)) AS min_ride_length
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned`
WHERE DATE_DIFF(ended_at, started_at, minute) > 0 
GROUP BY member_casual

# Count rides per weekday
SELECT
  COUNT(DISTINCT ride_id) AS num_of_rides,
  day_of_week,
  member_casual 
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned`
GROUP BY member_casual, day_of_week

# Average ride length per weekday
SELECT 
  AVG(ride_length) as avg_ride_length,
  day_of_week,
  member_casual
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned`
GROUP BY member_casual, day_of_week

# Count rides per month
SELECT
  COUNT(DISTINCT ride_id) AS num_of_rides,
  month,
  member_casual 
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned`
GROUP BY member_casual, month

# Average ride length per month
SELECT 
  AVG(ride_length) as avg_ride_length,
  month,
  member_casual
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned`
GROUP BY member_casual, month

# Create new time_of_day column, Britannica definitions of times of day
 CREATE OR REPLACE TABLE `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned` AS (
 SELECT
  *, 
  CASE 
  WHEN time BETWEEN '5:00:00' AND '11:59:59' THEN 'morning'
  WHEN time BETWEEN '12:00:00' AND '16:59:59' THEN 'afternoon'
  WHEN time BETWEEN '17:00:00' AND '20:59:59' THEN 'evening'
  WHEN time BETWEEN '21:00:00' AND '23:59:59' OR time BETWEEN '00:00:00' AND '4:59:59' THEN 'night'
  ELSE 'other'
  END AS time_of_day,
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned`
 )

# Count rides per time of day
SELECT
  COUNT(DISTINCT ride_id) AS num_of_rides,
  time_of_day,
  member_casual 
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned`
GROUP BY member_casual, time_of_day

# Average ride length per time of day
SELECT 
  AVG(ride_length) as avg_ride_length,
  time_of_day,
  member_casual
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned`
GROUP BY member_casual, time_of_day

# Most used start stations (10)
SELECT 
  start_station_name,
  COUNT(ride_id) AS num_of_rides
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned`
WHERE start_station_name IS NOT NULL
GROUP BY start_station_name
ORDER BY num_of_rides DESC
LIMIT 10

# Most used end stations (10)
SELECT 
  end_station_name,
  COUNT(ride_id) AS num_of_rides
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned`
WHERE end_station_name IS NOT NULL
GROUP BY end_station_name
ORDER BY num_of_rides DESC
LIMIT 10

# Count rides by rideable type
SELECT 
  COUNT(DISTINCT ride_id),
  rideable_type,
  member_casual
FROM `casestudy1-sql.2023_DivvyTripData.total_DivvyData_2023_cleaned`
GROUP BY rideable_type, member_casual
