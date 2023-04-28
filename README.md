# Case Study: Bellabeat Case Study
#### Author: Diego Castillejo


### ❓ [Ask](#1-ask)
### 💻 [Prepare](#2-prepare)
### 🛠 [Process](#3-process)
### 📊 [Analyze and Share](#4-analyze amd share)
### 🧗‍♀️ [Act](#6-act)



## Introduction
For my introduction I will be going through a brief description of the company this case study is focused on.  Bella beat is a tech manufacture with an array of health products curated toward women. Bella beat is described as a small but successful company with the potential to become a major competitor in the global smart device market. The company offers a multitude of products that are designed to collect and keep track of user data relating to daily activity such as:  sleep, exercise, step counts, and calories burned. In this study I will be designated as a junior analyst tasked with analyzing the data collected by Bella beat’s devices and make conclusions on how exactly consumers are using their products.  My overall goal will be to use these conclusions to better design the company’s products and help guide the company to new opportunities. 

## 1. Ask
**Business Task**

Analyze the dataset provided by Bella beat to identify current user trends to gain insight on its fitness wellness products. Our goal is to develop recommendations for how these trends can inform Bella beat marketing strategy.

**Key Stakeholders**

Urška Sršen : Bella beat’s cofounder and Chief Creative Officer
Sando Mur: Mathematician and Bella beat’s cofounder; key member of the Bella beat executive team
Bella beat marketing analytics team: A team of data analysts
Main Issues: 
1. What are some trends in smart device usage?
2. How could these trends apply to Bella beat customers?
3. How could these trends help influence Bella beat marketing strategy


## 2. Prepare
**DataSet Used**

FitBit Fitness Tracker Data (CC0: Public Domain, dataset made available through Mobius): This Kaggle data set
contains personal fitness tracker from thirty Fitbit users. Thirty eligible Fitbit users consented to the submission of
personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. It includes
information about daily activity, steps, and heart rate that can be used to explore users’ habits

The dataset can be found here: (https://www.kaggle.com/datasets/arashnic/fitbit)

**Data Description**

Datasets of personal data stored in 18 excel sheets. These sheets include values for physical activity, calories, step counts, and sleep monitoring. I have sorted the data by categories and timeframe of data; Daily; Hourly and Minute. Daily category includes activity, calories, steps, and intensities. Hourly folder includes calories, steps and intensities. Minute folder includes calories, intensities, MET, steps and sleep. I have also included a MISC folder that includes users weight logs and heart rate. 
I will be primary focused on the datasets labeled as: Daily Activity_merged, Daily Calories _merged, Daily Intensities_merged, Daily Steps_Merged, and Sleep Day_Merged
Data is dated from March 201 to May 2016. There4 is no current data
To check for the distinct user ids for each dataset I used both R and SQL. For R I used the command n_distinct and found: 33 users for daily activity, 33 for daily calories, 33 for daily intensities, 21 for sleep, and 8 for weight. I did the same for SQL using the Count(id) command and found the same number of user ids for each.  Verifying the number of users, I can say that the data is a limited but the central limit theorem allows for any conclusions I find to still be valid.  

## 3. Process

**Excel**

To start off my data cleaning process, I first used Microsoft Excel to look for duplicated and format the data to ensure the data was imported correctly to both R studio and Big Query (SQL), the two main tools I will be using for analysis:
Daily Activity_merged
1. Remove Duplicates: No duplicate values found
2.  Format Data: Format Column B (Activity Date) by Date;
3.  Format Numerical Data to two decimals

Daily Calories _merged
1. Remove Duplicates: No duplicate values found
2. Format Data: Format Column D (Activity Day) by Date;
3. Format Numerical Data to two decimals

Daily Intensities_merged
1. Remove Duplicates: No duplicate values found
2. Format Data in date format

Daily Steps_Merged
1. Remove Duplicates: No duplicate values found
1. Format Data in date format
Sleep Day_Merged
1. Remove Duplicates: 3 duplicate values removed, 410 unique values remain
2.  Format Data: Separated Hour and Date in columns,
3.  Format Activity Date and Hour by Format Tool

**RStudio**

For R studio I needed to process the data I order to develop graphs more easily. I first imported the files and assigned them a variable name:

dailyActivity <- read.csv("dailyActivity_merged.csv")
sleepDay <- read.csv("sleepDay_merged.csv")
hourlySteps <- read.csv("hourlySteps_merged.csv")
dailyIntensities <- read.csv(dailyIntensities.csc")

I then looked at to see if there was any duplicate or null values:

sum(is.na(dailyActivity))
sum(is.na(sleepDay))
sum(is.na(dailyIntensities))
sum(duplicated(dailyActivity))
sum(duplicated(sleepDay))
sum(duplicated(dailyIntensites))

After verifying duplicate values I went ahead and deleted them: 
sleep_day <- sleep_day[!duplicated(sleep_day), ]
 
To able to make graphs depending on certain days of the week I also added weekdays: 

daily_activity <- daily_activity %>% mutate( Weekday = weekdays(as.Date(ActivityDate, "%m/%d/%Y")))

Finally I merged daily activty and sleep day data and ordered the data Sunday to Monday to plot later:

merged1 <- merge(daily_activity,sleep_day,by = c("Id"), all=TRUE)
merged_data <- merge(merged1, weight, by = c("Id"), all=TRUE)

merged_data$Weekday <- factor(merged_data$Weekday, levels= c("Monday", 
    "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))


## 4. Analyze and Share
**User Activity Levels**

I will be starting my first half of my analysis using Big Query SQL. My first bit of code will be looking at the number of days that each unique user logs for their daily activity. This SQL query will categorize the use of each user and place them in their respective categories: 

```
SELECT
    Id,
    COUNT(Id) AS TotalUserActivity,
    CASE
        WHEN COUNT(Id) BETWEEN 21 AND 31 THEN 'High use'
        WHEN COUNT(Id) BETWEEN 11 AND 20 THEN 'Moderate use'
        WHEN COUNT(Id) BETWEEN 1 AND 10 THEN 'Minimal use'
    END AS type_of_use
FROM
    `BellaBeatData.dailyActivity`
GROUP BY Id
ORDER BY TotalUserActivity;
```
The query was able to show me that 29 users were active, 3 users were moderate, and 1 user was passive with logging their activity.
 
 ![image](https://user-images.githubusercontent.com/98061069/235008761-599f6715-4970-4936-bb06-6e8c61e0be12.png)


**Relationship between Total steps and Total distance **

After verifying the activity for users, it was time to start looking into the participants fitness habits. To begin I wanted to look at the relationship between the total steps and total distance that each user records.  The following query was able to show me how steps and distance were related: 
```
SELECT
    DISTINCT Id,
    SUM(TotalSteps) AS TotalSteps,
    SUM(TotalDistance) AS TotalDistance
FROM
    `BellaBeatData.dailyActivity`
GROUP BY Id
ORDER BY TotalDistance DESC;
```

This query was able to show me that the correlation between the total steps and total distance of each user was very positive. Suggesting that physically those who traveled longer distances did have higher step counts.  

![image](https://user-images.githubusercontent.com/98061069/235008790-2728c32e-d60b-4d3d-b926-e3ede7429fe4.png)

**Total Distance vs Total Calories**

When looking at the relationship of Total Steps and Total Distance it makes sense physically that those with higher step counts have higher total distances. However, when it comes to fitness goals this may not be as simple when looking deeper into a user’s habits. I wanted to see if the total distance had any sort of relationship with the total calories that each user burned. I used the following query to look at this relationship:
```
SELECT
    DISTINCT Id,
    SUM(TotalDistance) AS TotalDistance,
    SUM(Calories) AS TotalCalories 
FROM
    `BellaBeatData.dailyActivity`
GROUP BY Id
ORDER BY TotalCalories DESC;
```
![image](https://user-images.githubusercontent.com/98061069/235008827-83fbf96c-d94d-4191-bad7-d4562f01a555.png)


The query was able to show me that there is a slight positive correlation between the total distance a user logs versus the total calories they burn with the watch on. Looking at the numbers it is not as clear cut as the relationship between steps and distance. For example, the highest calories count belonged to a user that had about half the amount of distance logged compared to the second highest calorie burner. These values started point me toward the idea that higher stats for fitness goals don’t always lead to better outcomes. 
 



**Step Activity Levels**

Knowing that the habits of user may not be as straightforward as some data shows I wanted to take a look at how the users were doing with keeping up their daily steps. When it comes to step counts according to an article written on mayoclinicn.com doctors generally recommend at least 10,0000 steps per day. ( https://www.mayoclinic.org/healthy-lifestyle/fitness/in-depth/10000-steps/art-20317391#:~:text=It's%20a%20good%20idea%20to,a%20day%20every%20two%20weeks).Not everyone can realistically make this goal every day, but with this in mind I wanted to categorize the average number of steps by using 10,000 as a benchmark for “very active”. I will be using the following query to categorize step counts: 
```
SELECT 
ID,
AverageDailySteps,
CASE 
WHEN AverageDailySteps < 5000 then 'Sedentary'
WHEN AverageDailySteps >= 5000 and AverageDailySteps < 7500 then 'Lightly Active'
WHEN AverageDailySteps >=7500 and AverageDailySteps < 10000 then 'Moderately Active'
ELSE 'Very Active' END AS UserType
FROM
(SELECT
ID, SUM(TotalSteps)/ COUNT(ActivityDate) AS AverageDailySteps,
FROM
(SELECT
ID,
ActivityDate,
TotalSteps
FROM `BellaBeatData.dailyActivity` as DailyActivity
Order BY
ID,
ActivityDate
)
```
The query was able to show me that only 7 users were able to reach the goal of having on average 10,000 steps per day. 9 users had at least 7,500 steps, 9 users had at least 5,500 steps, and 8 users had under 5,000 steps. This query and the one above (showing me total calorie count) are able to give me an idea that steps are not the only factor in having decent calorie counts. 

![image](https://user-images.githubusercontent.com/98061069/235008856-edb9cf5e-6af7-4828-8ff2-740762e474d4.png)

 
**Activity vs Calories**

For my next analysis I decided to take my data to Rstudio which was a much better suited tool for me for my next inquiry. After taking a look at the average step counts for users I wanted to find the relationship between steps and calories as well as the sedentary number of minutes a user had on a certain day. I was able to do this with the following query: 

```
activeMinutesvsSteps <- ggplot(data = mergedData) + 
  geom_point(mapping=aes(x=TotalSteps, y=FairlyActiveMinutes), color = "maroon", alpha = 1/3) +
  geom_smooth(method = loess,formula =y ~ x, mapping=aes(x=TotalSteps, y=FairlyActiveMinutes, color=FairlyActiveMinutes), color = "maroon", se = FALSE) +
  geom_point(mapping=aes(x=TotalSteps, y=VeryActiveMinutes), color = "forestgreen", alpha = 1/3) +
  geom_smooth(method = loess,formula =y ~ x,mapping=aes(x=TotalSteps, y=VeryActiveMinutes, color=VeryActiveMinutes), color = "forestgreen", se = FALSE) +
  geom_point(mapping=aes(x=TotalSteps, y=LightlyActiveMinutes), color = "orange", alpha = 1/3) +
  geom_smooth(method = loess,formula =y ~ x,mapping=aes(x=TotalSteps, y=LightlyActiveMinutes, color=LightlyActiveMinutes), color = "orange", se = FALSE) +
   geom_point(mapping=aes(x=TotalSteps, y=SedentaryMinutes), color = "steelblue", alpha = 1/3) +
  geom_smooth(method = loess,formula =y ~ x,mapping=aes(x=TotalSteps, y=SedentaryMinutes, color=SedentaryMinutes), color = "steelblue", se = FALSE) + 
  annotate("text", x=35000, y=150, label="Very Active", color="black", size=3)+
  annotate("text", x=35000, y=50, label="Fairly Active", color="black", size=3)+
  annotate("text", x=35000, y=1350, label="Sedentary", color="black", size=3)+
  annotate("text", x=35000, y=380, label="Lightly  Active", color="black", size=3)+
  labs(x = "Total Steps", y = "Active Minutes", title="Steps vs Active Minutes")
```

This query was able to show me again that higher totals on fitness goals don’t always lead to the highest returns. Generally, the higher a person’s step count is the more calories they are likely to burn. However, from the graph below I can see that the highest calorie burner is not the user with the highest step count.  In fact, the person with the highest step total only had an average calorie count at best. Even looking at the majority of users they are still able to burn between 1500 to 2500 calories despite being under the 10,000-step goal that doctors recommend.  All of these values point to the idea that it when it comes to fitness goals the quality of the work you do outweighs the quantity of work. 
 

![image](https://user-images.githubusercontent.com/98061069/235009335-f0931535-2115-4535-88d1-d238ccf43fa1.png)



Ideas: watches could focus on providing alternatives for the user to complete health goals that would be more effective than increasing total step counts or longer distances. 

***Steps during Active Minutes***

To take a closer look at the idea that different forms of activity have different outcomes for fitness goals I decided to take a look at the relationship between intensity levels and calories burned using R. I was able to look at this relationship using the following query: 

```
ActiveMinutesvsCalories <- ggplot(data = mergedData) + 
  geom_point(mapping=aes(x=Calories, y=FairlyActiveMinutes), color = "maroon", alpha = 1/3) +
  geom_smooth(method = loess,formula =y ~ x, mapping=aes(x=Calories, y=FairlyActiveMinutes, color=FairlyActiveMinutes), color = "maroon", se = FALSE) +
  geom_point(mapping=aes(x=Calories, y=VeryActiveMinutes), color = "forestgreen", alpha = 1/3) +
  geom_smooth(method = loess,formula =y ~ x,mapping=aes(x=Calories, y=VeryActiveMinutes, color=VeryActiveMinutes), color = "forestgreen", se = FALSE) +  
  geom_point(mapping=aes(x=Calories, y=LightlyActiveMinutes), color = "orange", alpha = 1/3) +
  geom_smooth(method = loess,formula =y ~ x,mapping=aes(x=Calories, y=LightlyActiveMinutes, color=LightlyActiveMinutes), color = "orange", se = FALSE) +
  geom_point(mapping=aes(x=Calories, y=SedentaryMinutes), color = "steelblue", alpha = 1/3) +
  geom_smooth(method = loess,formula =y ~ x,mapping=aes(x=Calories, y=SedentaryMinutes, color=SedentaryeMinutes), color = "steelblue", se = FALSE) +
    annotate("text", x=4800, y=160, label="Very Active", color="black", size=3)+
  annotate("text", x=4800, y=0, label="Fairly Active", color="black", size=3)+
  annotate("text", x=4800, y=500, label="Sedentary", color="black", size=3)+
  annotate("text", x=4800, y=250, label="Lightly  Active", color="black", size=3)+
  labs(x = "Calories", y = "Active Minutes", title="Calories vs Active Minutes")
```


The query was able to show me a majority of the steps that were made by users we in a state of sedentary activity. Which would mean that when it comes to calories burned it is much more dependent on the types of activities that the user involves themselves I rather than keeping up step counts.  Looking at the four active levels to the total steps I saw most data is concentrated on users who take about 5000 to 15000 steps a day. These users spent an average between 8 to 13 hours in sedentary, 5 hours in lightly active, and 1 to 2 hours for fairly and very active.  
 
***Calories Vs Activity***

After taking a look at the relationship between activity and step counts, I wanted to do the same for calories and activity. This way I would be able to see how different levels of activity affect the calorie counts for users. I used the following query to do so: 

```
ActiveMinutesvsCalories <- ggplot(data = mergedData) + 
  geom_point(mapping=aes(x=Calories, y=FairlyActiveMinutes), color = "maroon", alpha = 1/3) +
  geom_smooth(method = loess,formula =y ~ x, mapping=aes(x=Calories, y=FairlyActiveMinutes, color=FairlyActiveMinutes), color = "maroon", se = FALSE) +
    geom_point(mapping=aes(x=Calories, y=VeryActiveMinutes), color = "forestgreen", alpha = 1/3) +
  geom_smooth(method = loess,formula =y ~ x,mapping=aes(x=Calories, y=VeryActiveMinutes, color=VeryActiveMinutes), color = "forestgreen", se = FALSE) +
   geom_point(mapping=aes(x=Calories, y=LightlyActiveMinutes), color = "orange", alpha = 1/3) +
  geom_smooth(method = loess,formula =y ~ x,mapping=aes(x=Calories, y=LightlyActiveMinutes, color=LightlyActiveMinutes), color = "orange", se = FALSE) +
    geom_point(mapping=aes(x=Calories, y=SedentaryMinutes), color = "steelblue", alpha = 1/3) +
  geom_smooth(method = loess,formula =y ~ x,mapping=aes(x=Calories, y=SedentaryMinutes, color=SedentaryeMinutes), color = "steelblue", se = FALSE) +
    annotate("text", x=4800, y=160, label="Very Active", color="black", size=3)+
  annotate("text", x=4800, y=0, label="Fairly Active", color="black", size=3)+
  annotate("text", x=4800, y=500, label="Sedentary", color="black", size=3)+
  annotate("text", x=4800, y=250, label="Lightly  Active", color="black", size=3)+
  labs(x = "Calories", y = "Active Minutes", title="Calories vs Active Minutes")
```


This query was able to show me that when it comes to calorie count it appears the majority of calories that are burned by users are during their sedentary hours, this may be accounting for the calories that the human body would burn to function normally.  According to the Medical New Today, the average man requires at least 2,700 calories per day to maintain bodyweight while the average woman needs 2,200 calories. From these daily calories around 60-75 percent go toward supplying energy to vital organs, muscles, and other bodily functions. Looking at the graph below I can see that the majority of calories burned fall under the sedentary range which does follow the logic of a good portion of calories being used for bodily functions. However, when I look at the end of the graph, I can also identify the upwards trend of calories burned when active and the downward trend of calories burned when not active.  This would suggest that the more a user burns calories while active cuts into the time they spend sedentary. 

![image](https://user-images.githubusercontent.com/98061069/235009590-6609c6ad-b997-4bfb-8c53-370e4b2695ce.png)


**Sleep Hours vs Calories**

After looking at calories burned during activity I wanted to see if there was any relationship between a user’s sleep quality and their calorie counts. I first wanted to look at the average number of hours of sleep for each user. According to the American Heart Association (https://www.heart.org/en/healthy-living/go-red-get-fit/sleep-women-and-heart-disease) doctors recommend a minimum of 6 hours of sleep.  I wanted to use SQL to see if users are able to average this threshold using the following query: 
```
SELECT
    Id,
    COUNT(CASE WHEN TotalMinutesAsleep > 360 THEN 1 ELSE NULL END) AS daysOver360Min,
    (AVG(TotalMinutesAsleep))/60 AS average_sleep_hours
FROM
    `BellaBeatData.sleepDay`
GROUP BY Id
HAVING daysOver360Min > 0;

This query was able to show me that of the users that did log sleep hours 16 of them did record at least 6 hours of sleep while 5 of them did not.
```
![image](https://user-images.githubusercontent.com/98061069/235009699-0466dac6-f781-45f1-bc45-a961faaa8f96.png)

After verifying that majority of the user were able to get adequate sleep I wanted to see if there was any relationship between sleep and calorie counts. For this I went ahead and used R with the following query: 

```
ggplot(data=mergedData, aes(x=TotalMinutesAsleep, y = Calories, color=TotalMinutesAsleep))+ 
     geom_point()+ 
     labs(title="Total Minutes Asleep vs Calories")+
     xlab("Total Minutes Alseep")+
     stat_smooth(method=lm)+
     scale_color_gradient(low="orange", high="steelblue")
```
The query was able to show me that there is no relationship between the hours a person sleeps and their calorie counts.

![image](https://user-images.githubusercontent.com/98061069/235009810-9892a3d7-9f7f-4a43-96d8-a0a1e1df9992.png)

## 5. Share
Recommendations
Recommending workout plans
There is a relationship between the activity that user does and the calories that they burn. The watch could lean toward giving user suggestion on exercises that may be more efficient for their fitness goals. 
Mixed Calories Counters
The watches have no function when it comes to distrusting calories burned during periods of activity. A feature should be added to be able to differentiate between the calories burned at a sedentary level and active level. 
Push Notifications on Mobile Device

Data some users are not logging as much activity as others, they could benefit from feature that allows the watches to remind users to log any activity through their phones. This way they will be more conscience of keeping track of progress.  


Progression System
The watch should have level system where users are notified with how far their progress is going in reaching their fitness goals.  For example, users can set a goal to have at least 5,000 steps per day and after a certain period of time achieving this, they can be pushed to go for 6,0000 to slowly progress. 

