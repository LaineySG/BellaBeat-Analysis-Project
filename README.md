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

From the above analysis of the mean values of each measured property, there are a few useful conclusions we can draw. 

* The tracker logged about 5.5km/day but people only logged 0.11km/day. This tells us the customers aren't diligent with logging km, or are underestimating their actual distance in a given day. From this we can see the value of the tracking device to provide an accurate representation of the distance travelled without needing to rely on the customer's logged information.

* The average distance and steps are 5.5km, and 7600 steps, respectively. A 2011 [study](https://ijbnpa.biomedcentral.com/articles/10.1186/1479-5868-8-79) found that healthy adults can take anywhere between approximately 4,000 and 18,000 steps/day, and that 10,000 steps/day is a reasonable target for healthy adults. This means that the average device user can step up their game, so to speak. The Bellabeat device can assist by providing reminders to walk instead of drive, or go for walks in the evening when possible.

* As shown in the python script below, 61% of activity is not very active and 27% is very active. 27% very active is quite good, but it would be better if more of the lightly active rate could be moderately active, as moderately active activity makes up only 10% of steps. The Bellabeat device could suggest things such as jogging instead of walking or taking the stairs in buildings to have more moderate/very active steps and less lightly active steps.

```very = round(1.5/5.49 * 100,2) #very active rate
mod = round(0.57/5.49 * 100,2) #moderately active rate
light = round(3.34/5.49 * 100,2) #lightly active rate
print("very active is", very,"%," " moderately active is", mod, "%,", " and lightly active is", light, "%, in terms of steps/kilometers.")```

* When we look at the data below of the time that the device user is not sedentary, about 85% is spent being lightly active. Very active is only 10%, or about 21 minutes per day. [Each week](https://www.cdc.gov/physicalactivity/basics/adults/index.htm) adults need 150 minutes of moderate-intensity physical activity, or about 30 mins per day as a minimum. This means that the device users are meeting the minimum requirements when considering the "very active" and "moderately active" time. This could always be improved through reminders and suggestions to the Bellabeat device user, however it shows that the users of smart devices are generally fairly active.

```very = round(21.16/(21.16+13.56+192.81) * 100,2) #very active rate, ignoring sedentary minutes
mod = round(13.56/(21.16+13.56+192.81) * 100,2) #moderately active rate, ignoring sedentary minutes
light = round(192.81/(21.16+13.56+192.81) * 100,2) #lightly active rate, ignoring sedentary minutes
print("very active is", very,"%," " moderately active is", mod, "%,", " and lightly active is", light, "%, in terms of minutes of exercise.")```

* The final observation from this data is that the average daily calories is 2300 per day. [Recommended daily calorie intakes](https://www.medicalnewstoday.com/articles/245588#_noHeaderPrefixedContent) in the US are around 2,500 for men and 2,000 for women. As the data doesn't specify the sex of the participants, if we assume them to be about 50% male and 50% female then this number seems spot on. The Bellabeat device could offer dietary assistance such as calorie tracking for users, especially those with weight gain/loss goals. 

```SELECT MAX(Calories) FROM Bellabeat.daily_activity
#>>> Results: 4900
SELECT Calories FROM Bellabeat.daily_activity ORDER BY Calories DESC
#>>> Results shown below```

![image](https://user-images.githubusercontent.com/106799436/171965973-6b944f98-a28b-4435-9c07-545c686eb576.png)

Due to the number of extremely high calories I decided to dive further into this. I ran the below function to determine if there was an explanation for the high caloric intakes based on high user activity on a given day.

```SELECT AVG(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes) AS AvgMins FROM Bellabeat.daily_activity;
#>>> Results: 227.54 mins (This is the average active minutes of all people.)
WITH Summins AS (select (VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes) AS summins, Calories from Bellabeat.daily_activity)
SELECT AVG(Summins.summins) as Avgmins  FROM Summins WHERE Calories > 3000;
#>>> Results: 309.75816993464059 (This is the average active minutes of people who consumed >3000 calories on a given day.)```

I also ran this script for calories under 1200, and found an average of 32 mins of activity vs 230 minutes for the average person eating ~2300 calories per day. 32 minutes of activity is a fairly sedentary lifestyle but under 1200 calories is still unlikely to make up the difference in active minutes. I believe that this, as well as the information learned from those eating over 3000 calories per day highlights the usefulness to dietary assistance, tips, and tracking using Bellabeat products. 

Next I looked at the sleep patterns of device users.

```SELECT ROUND(AVG(TotalMinutesAsleep),2) AS AvgTotalMinutesAsleep, ROUND(AVG(TotalTimeInBed),2) AS AvgTotalTimeInBed FROM Bellabeat.daily_sleep;
#>>> Results: AvgTotalMinutesAsleep: 419.17 (7 hours per night)
#>>> Results: AvgTotalTimeInBed: 458.48 (38 mins)

SELECT COUNT(*) FROM Bellabeat.daily_sleep WHERE TotalMinutesAsleep < 420;
#>>> Results: 181 (Users who received less than 7 hours of sleep)
SELECT COUNT(*) FROM Bellabeat.daily_sleep;
#>>> Results: 410 (Total tracked sleeps)

SELECT AVG(TotalTimeInBed) FROM Bellabeat.daily_sleep WHERE TotalSleepRecords > 1;
#>>> Results: 512 (8.5 hours of sleep for those who slept more than once, or napped during the day) 

SELECT AVG(TotalTimeInBed - TotalMinutesAsleep) AS SleepDiff FROM Bellabeat.daily_sleep;
#>>> Results: 39.30 mins (Average time taken to fall asleep + get out of bed in the morning.)```

The above data shows that the average device user is getting 7 hours per night of sleep. That is lower than ideal, as 45% of tracked sleeps are under the [7 hour recommended healthy sleep minimum](https://www.sleepfoundation.org/how-sleep-works/how-much-sleep-do-we-really-need#:~:text=National%20Sleep%20Foundation%20guidelines1,to%208%20hours%20per%20night.). The Bellabeat device can assist with this through reminders to set a reasonable time to go to bed each night to ensure that they are getting a reasonable amount of sleep each night, preferably between 7 and 9 hours. Another thing to note is that users who have more than one sleep record got more sleep than users who only slept once. This could indicate that napping leads to more sleep, and could be a healthy recommendation if possible for the user. It could also be that these users are waking up in the middle of the night and sleeping longer to make up for it.

```SELECT ROUND(AVG(BMI),2) FROM Bellabeat.daily_weight_log;
#>>> Results: 25.19 (Average BMI)

SELECT COUNT(*) FROM Bellabeat.daily_weight_log WHERE BMI > 24.99;
#>>> 33

SELECT COUNT(*) FROM Bellabeat.daily_weight_log WHERE BMI < 18.5;
#>>> 0

SELECT COUNT(*) FROM Bellabeat.daily_weight_log;
#>>> 67```

[A healthy BMI range is between 18.5 and 24.9.](https://www.nhs.uk/common-health-questions/lifestyle/what-is-the-body-mass-index-bmi/#:~:text=BMI%20ranges&text=below%2018.5%20%E2%80%93%20you're%20in,re%20in%20the%20obese%20range) The average BMI of the participants in this data is 25.19, which is slightly higher than the healthy range. 49% of participants measured over 24.99 BMI, and zero measured under 18.5. This shows that it is best for the Bellabeat app to focus heavily on weight loss. 

![image](https://user-images.githubusercontent.com/106799436/171966059-4c1e6ad2-c04e-4864-a83b-ad1e4ac2c394.png)

I entered the data into Tableau and joined it by ID and Date to do some further analysis. I started with the weight and time data that I was just looking at. Most of the data is from manual reports rather than automated reports. Of the data we have, 3 people gained weight and 2 people lost weight. Unfortunately this isn't a lot of data to go off of, as only 5 users entered more than one data point. It does corroborate the point that weight loss should be the focus of the Bellabeat devices. 

![image](https://user-images.githubusercontent.com/106799436/171966077-eeb3e699-090e-4e09-9ef9-4bf1acdf371c.png)

As seen above, only on Saturdays, Sundays, and Wednesdays do users get over 8 hours of sleep on average. It would be beneficial for the device to send reminders to users on other weekdays to bring that higher, and encourage users to go to bed earlier in the evening. As previously stated, even if the average is 7 hours, that means that half of the users are likely to be below 7 hours of sleep per night assuming realistic levels of variety in the data.

![image](https://user-images.githubusercontent.com/106799436/171966102-be00ba84-1186-4f2b-833b-95f48ac85241.png)

The average user is getting an extremely variable amount of calories. Ideally the daily calories would be closer to the grey band, between 2000 and 2500 calories depending on size and gender of the user. The Bellabeat app could assist with caloric tracking to ensure that users are able to keep to a more stable and healthy caloric range.


![image](https://user-images.githubusercontent.com/106799436/171966115-e026a57d-ca62-4b64-83ab-4e77f42f8d4c.png)

According to the above graph, most steps take place around lunch, and presumably after work between 5 and 7 PM. The Bellabeat software could remind users to try to put in some steps either before work, or later in the evening when there are lulls in activity. 

![image](https://user-images.githubusercontent.com/106799436/171966139-3fbc481c-dfbc-4de5-a7ed-13b6dfca62d6.png)

As shown above, there are days with a high rate of sedentary minutes as well as high average calorie intake, such as Saturdays and Tuesdays. Friday also has a higher caloric rate with higher sedentary minutes. The Bellabeat app could assist the user in reminding them to balance their caloric intake and their activity. A higher activity rate means a higher caloric requirement and vice versa. 

![image](https://user-images.githubusercontent.com/106799436/171966158-626f7e07-9b16-4f8a-901d-25b3bd59c59e.png)

Users intensity is at it's highest around lunchtime and 6 or 7 in the evening, presumably after getting home from work. Bellabeat can capitalize on this trend by reminding users to take advantage of this time for exercise. 

![image](https://user-images.githubusercontent.com/106799436/171966173-124ed4eb-0ffc-4ae2-923c-50117fc1c375.png)



Most calories are consumed around lunch and between 5 and 7 PM. This coincides with the highest intensity periods, showing that people tend to be eating and exercising primarily around these time periods. It would be helpful for the Bellabeat app to focus any weight-loss reminders around these times when people are eating or exercising. 

You can find a copy of all visualizations linked [here, on Tableau.](https://public.tableau.com/app/profile/lain4377/viz/BellaBeatcasestudy/Dashboard1#1)



Throughout this project we have seen that there are many ways for Bellabeat to get more value out of their marketing for both the user as well as the business. Some highlights for potential upgrades and changes include: 

* We should ensure that the Bellabeat device provides an accurate representation of the distance travelled without needing to rely on the customer's logged information.

* The Bellabeat device can assist by providing alternative suggestions such as to walk instead of drive, jog instead of walk, take the stairs in buildings instead of elevators, or go for walks in the evening when possible in order to increase user activity.

* The Bellabeat device can offer dietary assistance such as calorie tracking for users, especially those with weight gain/loss goals. 

* The Bellabeat device can assist with reminders to set a reasonable time to go to bed each night to ensure that they are getting a reasonable amount of sleep each night, preferably between 7 and 9 hours. Napping could be a useful suggestion for users to increase amounts of sleep.

* Only on Saturdays, Sundays, and Wednesdays do users get over 8 hours of sleep on average. It would be beneficial for the device to send reminders to users on other weekdays to bring that higher, and encourage users to go to bed earlier in the evening.

* The average user is getting an extremely variable amount of calories. Ideally the daily calories would be between 2000 and 2500 calories depending on size and gender of the user. The Bellabeat app should assist with caloric tracking to ensure that users are able to keep to a more stable and healthy caloric range.

* The Bellabeat device can remind users to try to put in some steps either before work, or later in the evening when there are lulls in activity. 

* There are days with a high rate of sedentary minutes as well as high average calorie intake, such as Saturdays, Tuesdays, and Fridays. The Bellabeat app could assist the user in reminding them to balance their caloric intake and their activity. 

* Users intensity is at it's highest around lunchtime and 6 or 7 in the evening, presumably after getting home from work. Bellabeat should capitalize on this trend by reminding users to take advantage of this time for exercise. 

* The Bellabeat app should focus any weight-loss reminders around lunchtime and between 5 and 7 PM when people are eating and exercising.































