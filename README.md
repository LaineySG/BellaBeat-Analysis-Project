# BellaBeat-Analysis-Project
Worked with real-life data for a case study on improving marketing and customer experience for Bellabeat. Asked questions pertaining to the business task, prepared, processed, and cleaned data. Analyzed the data using SQL and visualizations. Imported and processed data in Tableau to further analyze and visualize the data, presenting tangible and actionable suggestions for improving customer experience and Bellabeat marketing.


**Task:** To analyze data on smart device usage, and provide insights on how to improve marketing for a chosen Bellabeat device.

**Major Questions:**

1) Identify trends in smart device usage patterns and customers

2) Identify how to apply this information to Bellabeat customers

3) Identify how to apply this information to Bellabeat marketing strategies

**Deliverables:**

1) Clear summary of business task

2) Description of data sources used

3) Data cleaning/manipulation documentation

4) Analysis summary

5) Visualizations and key findings

6) High level marketing recommendations

**Data used:**

The data used for this analysis is from the [FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit?datasetId=1041311&searchQuery=sql), it is public domain data made available through Morbius and accessed through Kaggle. This data set contains personal fitness tracker information from thirty fitbit users who consented to submission of their data. There is no user-specific data which may lead to privacy concerns, and no clear bias or issues with the data.

**Preliminary cleaning:**

1) Used left() function in excel to keep only date from sleepDay_merged.csv, weightLogInfo_merged, heartrate_seconds_merged, hourlyCalories_merged, hourlyIntensities_merged, hourlySteps_merged as the AM/PM and time confused bigquery data import

2) imported all data into BigQuery SQL


```
#Processing data: Ran the following script on Bigquery to clean and process the data to be ready for analysis, after importing the CSV-format data onto bigquery. 
SELECT
COUNT(DISTINCT Id), COUNT(DISTINCT ActivityDate) FROM Bellabeat.daily_activity; 
#>>>Results: 33,31
#Ran this script for most daily logs to count distinct ID and Days to ensure data integrity. 
#Should be 33, 31 respectively as there were 33 participants and 31 days logged.

SELECT
COUNT(DISTINCT Id), COUNT(DISTINCT ActivityDay) FROM Bellabeat.daily_calories;
#>>>Results: 33,31

SELECT
COUNT(DISTINCT Id), COUNT(DISTINCT ActivityDay) FROM Bellabeat.daily_intensities;
#>>>Results: 33,31

SELECT
COUNT(DISTINCT Id), COUNT(DISTINCT SleepDay) FROM Bellabeat.daily_sleep;
#>>>Results: 24,31
#results differed for number of distinct IDs due to only 24 of the 33 users either consenting to sleep data tracking, or only 24 having the means to track
#sleep patterns due to equipment used. This doesn't affect data integrity, but does mean that sleep patterns have less data than other metrics. 

SELECT
COUNT(DISTINCT Id), COUNT(DISTINCT ActivityDay) FROM Bellabeat.daily_steps;
#>>>Results: 33,31

SELECT
COUNT(DISTINCT Id), COUNT(DISTINCT Date) FROM Bellabeat.daily_weight_log;
#>>>Results: 8,31
#results differed for number of distinct IDs due to only 8 of the 33 users either consenting to weight data tracking, or only 8 having the means to track
#weight patterns due to equipment used. This doesn't affect data integrity, but does mean that weight patterns have much less data than other metrics. 

#For the below 4 queries, the date format was converted (back) to datetime from separate date and time columns. These were split prior to importing due to
#aformentioned issues with bigquery import. The hour and second-wise data will be easier to work with using the datetime format. 

SELECT
Day, Time, (SELECT COUNT(DISTINCT Id) FROM Bellabeat.heartrate_seconds),
(SELECT COUNT(DISTINCT Day) FROM Bellabeat.heartrate_seconds), 
PARSE_DATETIME('%Y-%m-%d %H:%M:%S %p', (SELECT CONCAT(Day," ", Time) AS DayTime)) AS datetime FROM Bellabeat.heartrate_seconds;
#>>>Results: 7,31 (results are ignoring day, time, and datetime columns)
#results differed for number of distinct IDs due to only 7 of the 33 users either consenting to heartrate data tracking, or only 7 having the means to track
#heartrate patterns due to equipment used. This doesn't affect data integrity, but does mean that heartrate patterns have much less data than other metrics. 

SELECT Day, Time, (SELECT COUNT(DISTINCT Id) FROM Bellabeat.hourly_calories),
(SELECT COUNT(DISTINCT Day) FROM Bellabeat.hourly_calories),
PARSE_DATETIME('%Y-%m-%d %H:%M:%S %p', (SELECT CONCAT(Day," ", Time) AS DayTime)) AS datetime FROM Bellabeat.hourly_calories;
#>>>Results: 33,31 (results are ignoring day, time, and datetime columns)

SELECT date_field_1, string_field_2, (SELECT COUNT(DISTINCT Id) FROM Bellabeat.hourly_intensities),
(SELECT COUNT(DISTINCT date_field_1) FROM Bellabeat.hourly_intensities),
PARSE_DATETIME('%Y-%m-%d %H:%M:%S %p', (SELECT CONCAT(date_field_1," ", string_field_2) AS DayTime)) AS datetime FROM Bellabeat.hourly_intensities;
#>>>Results: 33,31 (results are ignoring day, time, and datetime columns)

SELECT Day, Time, (SELECT COUNT(DISTINCT Id) FROM Bellabeat.hourly_steps),
(SELECT COUNT(DISTINCT Day) FROM Bellabeat.hourly_steps),
PARSE_DATETIME('%Y-%m-%d %H:%M:%S %p', (SELECT CONCAT(Day," ", Time) AS DayTime)) AS datetime FROM Bellabeat.hourly_steps;
#>>>Results: 33,31 (results are ignoring day, time, and datetime columns)

#I then wanted to check for duplicate entries. I ran the following code to do so, beginning with the hourly_steps column.
SELECT Id, Day, Time, StepTotal from Bellabeat.hourly_steps 
GROUP BY Id, Day, Time, StepTotal
HAVING COUNT(Id) > 1 AND COUNT(Day) > 1 AND COUNT(Time) > 1;
#>>> Results: There is no data to display.
#I wasn't sure if the script ran correctly so I decided to add a duplicate entry manually, and ensure it would catch it.

--INSERT INTO Bellabeat.hourly_steps
--(Id, Day, Time, StepTotal)
--VALUES (6117666160,  '2016-04-28',  '11:00:00 PM',  0);
#>>> Results: This statement added 1 row to hourly_steps.
# After successfully adding the duplicate row, I ran the duplicate checking script again and got the following.
#>>> Results: 6117666160 2016-04-28 11:00:00 PM 0
#This tells us that it did find a duplicate row, as predicted. Finally, I ran the below code to delete the duplicate entries, then ran the INSERT INTO code to add
#a single instance of the row back in. 

--DELETE FROM Bellabeat.hourly_steps WHERE Id=6117666160 AND Day='2016-04-28' AND Time = '11:00:00 PM';
#>>> This statement removed 2 rows from hourly_steps.

--INSERT INTO Bellabeat.hourly_steps
--(Id, Day, Time, StepTotal)
--VALUES (6117666160,  '2016-04-28',  '11:00:00 PM',  0);
#>>> This statement added 1 row to hourly_steps.

#I then ran this for all tables to look for duplicates

SELECT Id, ActivityDate from Bellabeat.daily_activity 
GROUP BY Id, ActivityDate
HAVING COUNT(Id) > 1 AND COUNT(ActivityDate) > 1;
#>>> Results: There is no data to display.

SELECT Id, ActivityDay from Bellabeat.daily_calories 
GROUP BY Id, ActivityDay
HAVING COUNT(Id) > 1 AND COUNT(ActivityDay) > 1;
#>>> Results: There is no data to display.

SELECT Id, ActivityDay from Bellabeat.daily_intensities 
GROUP BY Id, ActivityDay
HAVING COUNT(Id) > 1 AND COUNT(ActivityDay) > 1;
#>>> Results: There is no data to display.

SELECT Id, SleepDay from Bellabeat.daily_sleep
GROUP BY Id, SleepDay
HAVING COUNT(Id) > 1 AND COUNT(SleepDay) > 1;
#>>> Results: 4388161847 2016-05-05  
#>>> Results: 4702921684 2016-05-07
#>>> Results: 8378563200 2016-04-25
#This returned 3 duplicate rows. I wanted to double check that they were perfect duplicates so I ran the following:

SELECT * FROM Bellabeat.daily_sleep WHERE Id = 4388161847 AND SleepDay = '2016-05-05';
#>>> Results: 4388161847 2016-05-05 1 471 495
#>>> Results: 4388161847 2016-05-05 1 471 495
SELECT * FROM Bellabeat.daily_sleep WHERE Id = 4702921684 AND SleepDay = '2016-05-07';
#>>> Results: 4702921684 2016-05-07 1 520 543
#>>> Results: 4702921684 2016-05-07 1 520 543
SELECT * FROM Bellabeat.daily_sleep WHERE Id = 8378563200 AND SleepDay = '2016-04-25';
#>>> Results: 8378563200 2016-04-25 1 388 402
#>>> Results: 8378563200 2016-04-25 1 388 402

#After confirming that these are duplicate rows, I deleted one of the instances of each of the three rows. 

DELETE FROM Bellabeat.daily_sleep WHERE Id=4388161847 AND SleepDay='2016-05-05' AND TotalTimeInBed = 495;
#>>> Results: This statement removed 2 rows from daily_sleep.
INSERT INTO Bellabeat.daily_sleep
(Id,	SleepDay,	TotalSleepRecords,	TotalMinutesAsleep,	TotalTimeInBed)
VALUES (4388161847, '2016-05-05', 1, 471, 495);
#>>> Results: This statement added 1 row to daily_sleep.
DELETE FROM Bellabeat.daily_sleep WHERE Id=4702921684 AND SleepDay='2016-05-07' AND TotalTimeInBed = 543;
#>>> Results: This statement removed 2 rows from daily_sleep.
INSERT INTO Bellabeat.daily_sleep
(Id,	SleepDay,	TotalSleepRecords,	TotalMinutesAsleep,	TotalTimeInBed)
VALUES (4702921684, '2016-05-07', 1, 520, 543);
#>>> Results: This statement added 1 row to daily_sleep.
DELETE FROM Bellabeat.daily_sleep WHERE Id=8378563200 AND SleepDay='2016-04-25' AND TotalTimeInBed = 402;
#>>> Results: This statement removed 2 rows from daily_sleep.
INSERT INTO Bellabeat.daily_sleep
(Id,	SleepDay,	TotalSleepRecords,	TotalMinutesAsleep,	TotalTimeInBed)
VALUES (8378563200, '2016-04-25', 1, 388, 402);
#>>> Results: This statement added 1 row to daily_sleep.
#I then ran the search query again to ensure only 1 row was remaining. 

SELECT * FROM Bellabeat.daily_sleep WHERE Id = 4388161847 AND SleepDay = '2016-05-05';
#>>> Results: 4388161847 2016-05-05 1 471 495
SELECT * FROM Bellabeat.daily_sleep WHERE Id = 4702921684 AND SleepDay = '2016-05-07';
#>>> Results: 4702921684 2016-05-07 1 520 543
SELECT * FROM Bellabeat.daily_sleep WHERE Id = 8378563200 AND SleepDay = '2016-04-25';
#>>> Results: 8378563200 2016-04-25 1 388 402

SELECT Id, ActivityDay from Bellabeat.daily_steps 
GROUP BY Id, ActivityDay
HAVING COUNT(Id) > 1 AND COUNT(ActivityDay) > 1;
#>>> Results: There is no data to display.

SELECT Id, Date, Time from Bellabeat.daily_weight_log 
GROUP BY Id, Date, Time
HAVING COUNT(Id) > 1 AND COUNT(Date) > 1 AND COUNT(Time) > 1;
#>>> Results: There is no data to display.

SELECT Id, Day, Time from Bellabeat.heartrate_seconds 
GROUP BY Id, Day, Time
HAVING COUNT(Id) > 1 AND COUNT(Day) > 1 AND COUNT(Time) > 1;
#>>> Results: There is no data to display.

SELECT Id, Day, Time from Bellabeat.hourly_calories 
GROUP BY Id, Day, Time
HAVING COUNT(Id) > 1 AND COUNT(Day) > 1 AND COUNT(Time) > 1;
#>>> Results: There is no data to display.

SELECT Id, date_field_1, string_field_2 from Bellabeat.hourly_intensities 
GROUP BY Id, date_field_1, string_field_2
HAVING COUNT(Id) > 1 AND COUNT(date_field_1) > 1 AND COUNT(string_field_2) > 1;
#>>> Results: There is no data to display.

SELECT Id, Day, Time from Bellabeat.hourly_steps 
GROUP BY Id, Day, Time
HAVING COUNT(Id) > 1 AND COUNT(Day) > 1 AND COUNT(Time) > 1;
#>>> Results: There is no data to display.

#After checking for distinct values and duplicates, I went on to order the rows by each column in both ascending and descending order to check for outliers.
#I will show a couple of examples below but not all of the script as it is a lot of queries. I found no significant outliers or errors. I did find some users
#who seemingly spent the entire day sedentary, ate 0 calories, and took 0 steps. I left these in as they don't appear to be errors. A likely explanation is that
#the user either forgot to wear the tracking device on this day, or possibly stayed in bed all day. Either way it is potentially useful data to analyze.

SELECT * 
FROM Bellabeat.daily_activity 
ORDER BY Id ASC;

SELECT * 
FROM Bellabeat.daily_activity 
ORDER BY Id DESC;

#After looking through for outliers in the data, I looked over the data types and they were fine. Finally I ran some IF_NULL script to see if there were any issues
#with missing data.

SELECT * FROM Bellabeat.daily_activity WHERE Id IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE ActivityDate IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE TotalSteps IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE TotalDistance IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE TrackerDistance IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE LoggedActivitiesDistance IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE VeryActiveDistance IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE ModeratelyActiveDistance IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE LightActiveDistance IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE SedentaryActiveDistance IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE VeryActiveMinutes IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE FairlyActiveMinutes IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE LightlyActiveMinutes IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE SedentaryMinutes IS NULL;
SELECT * FROM Bellabeat.daily_activity WHERE Calories IS NULL;
SELECT * FROM Bellabeat.daily_sleep WHERE Id IS NULL OR SleepDay IS NULL OR TotalSleepRecords IS NULL OR TotalMinutesAsleep IS NULL OR TotalTimeInBed IS NULL;
#All of the above returned no rows. I repeated this for all tables and rows. 

--SELECT * FROM Bellabeat.daily_weight_log WHERE Id IS NULL OR Date IS NULL OR Time IS NULL OR WeightKg IS NULL OR WeightPounds IS NULL OR Fat IS NULL OR BMI IS NULL OR IsManualReport IS NULL OR LogId IS NULL;
--SELECT * FROM Bellabeat.daily_weight_log WHERE Fat IS NULL;
#In the daily_weight_log table, the "Fat" column is Null except in 2 cases. There is 
#no documentation on the meaning of this but I would hypothetize that this is body fat %, and only those two logs had either the equipment to measure the body
#fat percentage, or they only measured these 2 times. I decided to delete this column as only 2/67 data entries is not enough to draw an accurate conclusion.
SELECT * FROM Bellabeat.daily_weight_log;
ALTER TABLE Bellabeat.daily_weight_log
DROP COLUMN Fat;
#>>> Result: This statement altered the table named daily_weight_log.

#With this, I was happy that the data was clean and processed. ```

**ANALYSIS**


```with Activity AS (select * from Bellabeat.daily_activity
ORDER BY ActivityDate DESC)

SELECT * FROM Activity AS activity
JOIN Bellabeat.daily_calories AS calories
ON activity.Id = calories.Id AND activity.ActivityDate = calories.ActivityDay
ORDER BY activity.ActivityDate DESC;

#I ran the above script to create a temp table and join the daily_calories and daily_activity tables. It looks like the data from daily_calories is
#already in daily_activity. This is good to know for the analysis process.

SELECT ROUND(AVG(TrackerDistance),2) AS TrackerDistance, ROUND(AVG(LoggedActivitiesDistance),2) AS LoggedActivitiesDistance, ROUND(AVG(VeryActiveDistance),2) AS VeryActiveDistance,
ROUND(AVG(ModeratelyActiveDistance),2) AS ModeratelyActiveDistance, ROUND(AVG(LightActiveDistance),2) as LightActiveDistance, ROUND(AVG(SedentaryActiveDistance),2) as SedentaryActiveDistance,
ROUND(AVG(SedentaryActiveDistance),2) as SedentaryActiveDistance, ROUND(AVG(VeryActiveMinutes),2) as VeryActiveMinutes, ROUND(AVG(FairlyActiveMinutes),2) as FairlyActiveMinutes,
ROUND(AVG(LightlyActiveMinutes),2) as LightlyActiveMinutes, ROUND(AVG(SedentaryMinutes),2) as SedentaryMinutes,
ROUND(AVG(TotalSteps),2) as TotalSteps, ROUND(AVG(TotalDistance),2) as TotalDistance, ROUND(AVG(Calories),2) as Calories
FROM Bellabeat.daily_activity
#>>> Result: See below```

![image](https://user-images.githubusercontent.com/106799436/171965640-3038ca6e-e5bd-48c8-b073-bf3e62f5f098.png)







