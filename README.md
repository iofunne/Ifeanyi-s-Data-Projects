---
title: "Bellabeats Case Study"
author: "Ifeanyi Ofunne"
date: "`r Sys.Date()`"
output: pdf_document
---

# Projects Overview

This is one of some personal projects I've undertaken in the purpose of enhancing/showcasing my data science skills and analytical processes. Enjoy :)

## Introduction and background

In this case study, I will make data driven recommendations for Bellabeat, a high-tech manufacturer of health-focused products for women. I analyze smart device data to gain insight into how consumers use their smart devices and provide insights on how to improve the features offered on their product.

### Area of focus

I chose to focus on the Leaf bracelet, which is smart jewlery that works as a wellness tracker that collects information such as: fitness activity, sleep habits, and menstrual cycles.

## Data integrity

The data presented was generated by respondents (on a Fitbit) to a distributed survey via Amazon Mechanical Turk between 03.12.2016-05.12.2016. Eligible Fitbit users consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. Variation between output represents use of different types of Fitbit trackers and individual tracking behaviors / preferences. This dataset was pulled from [Kaggle](https://www.kaggle.com/arashnic/fitbit) and was made available through Mobius.

The original data source can be found [here](https://zenodo.org/record/53894#.Y7nl-XbMK01).The metadata confirms it is open-source. The owner has dedicated the work to the public domain by waiving all of his or her rights to the work worldwide under copyright law, including all related and neighboring rights, to the extent allowed by law. The data can be copied, modified, distributed without asking permission.

### Dataset

There are 18 different CSV files included in the dataset.

### Loading common packages and data landscape

Loading tidyverse and other relevant packages into R for analysis.

```{r}
install.packages(c("tidyverse","readxl", "lubridate", "skimr", "timetk"), repo = "http://lib.stat.cmu.edu/R/CRAN/")
library("lubridate")
library("tidyverse")
library("readxl")
library("skimr")
library("timetk")
```

### Loading/Viewing data files

First, we created numerous data frames by reading in the following XLSX files created from the original data set. Note, these files were converted to an excel format after downloading.

```{r}
# load and store all relevant xlsx files
fb_daily_activity <- read_excel("Downloads/Fitabase Data 4.12.16-5.12.16 2/dailyActivity_merged.xlsx")
fb_hourly_int <- read_excel("Downloads/Fitabase Data 4.12.16-5.12.16 2/hourlyIntensities_merged.xlsx")
fb_minutes_int_2 <- read_excel("Downloads/Fitabase Data 4.12.16-5.12.16 2/minuteIntensitiesWide_merged.xlsx")
fb_minutes_asleep <- read_excel("Downloads/Fitabase Data 4.12.16-5.12.16 2/minuteSleep_merged.xlsx")
fb_day_asleep <- read_excel("Downloads/Fitabase Data 4.12.16-5.12.16 2/sleepDay_merged.xlsx")
fb_minutes_cal_2 <- read_excel("Downloads/Fitabase Data 4.12.16-5.12.16 2/minuteCaloriesWide_merged.xlsx")

# load csv files - to prevent data loss
fb_minutes_int <- read_csv("Downloads/Fitabase Data 4.12.16-5.12.16 2/minuteIntensitiesNarrow_merged.csv")
fb_minutes_cal <- read_csv("Downloads/Fitabase Data 4.12.16-5.12.16 2/minuteCaloriesNarrow_merged.csv")
```

### Checking and cleaning data
Then we use the head function to observe the shape and characteristics of the datasets which we'll then break down further for analysis.

```{r}
head(fb_hourly_int)
head(fb_daily_activity)
head(fb_minutes_asleep)
head(fb_day_asleep)
head(fb_minutes_int)
head(fb_minutes_cal)
```

The head function prompts us to update the data type for the date/datetime column for all data frames. Moreover, there is uncertainty in the unit of measure for several of the columns such as "TotalIntensity", "logId" or "value". We will transform the data and compare the granular minute/hourly data files with the their aggregated counterparts to check for consistency.

Note: A preliminary analysis on the sleep minutes excel spreadsheet showed that the "value" column most likely represents minutes spent in bed. That said, we can see that the data for this particular file might have been corrupted or tampered with. Let us assume, for the sake of argument, that the values should all represent one minute respectively.

```{r}
#transforming date fields to datetime
fb_minutes_asleep %>%
  mutate(date = as.POSIXct(date, tz = sys.timezone(), format = "%m/%d/%Y %H:%M"))

fb_day_asleep %>%
  mutate(SleepDay=as.Date(SleepDay, format = "%m/%d/%Y"))

fb_hourly_int %>%
  mutate(ActivityHour = as.POSIXct(ActivityHour, format = "%m/%d/%Y %H:%M"))

fb_minutes_int %>%
  mutate(ActivityMinute = as.POSIXct(ActivityMinute, format =  "%H:%M:%S %p"))

fb_minutes_cal %>%
  mutate(ActivityMinute = as.POSIXct(ActivityMinute, format = "%m/%d/%Y %H:%M:%S %p"))

fb_daily_activity %>%
  mutate(ActivityDate = as.Date(ActivityDate, format = "%m/%d/%Y"))
```
Lastly, we want to transform the wide datasets for the caloric and intensity data to a long format for easy manipulation later down the line. We can assume that the columns are minute by minute records of the data metric seeing as there are a total of 60 columns for each hour. We want to pivot these columns 

```{r}
fb_minutes_int_2 <- fb_minutes_int_2 %>%
  pivot_longer(cols = starts_with("Intensity"), names_to = "minute", names_prefix = "Intensity", values_to = "Intensity_record")
head(fb_minutes_int_2)

fb_minutes_cal_2 <- fb_minutes_cal_2 %>%
  pivot_longer(cols = starts_with("Calories"), names_to = "minute", names_prefix = "Calories", values_to = "Calories_burned")
head(fb_minutes_cal_2)
```
The wide data sets appear to be missing dates that were found in the aggregated data so we will disregard for now.

### Comparing datasets
The skimr package lends the n_unique fuction which tell us how many unique identifiers are in the dataset while the distinct and drop_na fuctions will remove any duplicates and missing data fields.

```{r}
# confirming Id count for related data sets
n_unique(fb_daily_activity$Id) == n_unique(fb_minutes_cal$Id) 
n_unique(fb_hourly_int$Id) == n_unique(fb_minutes_int$Id)
n_unique(fb_minutes_asleep$Id) == n_unique(fb_day_asleep$Id)

# drop all null values
fb_day_asleep <- fb_day_asleep %>%
  distinct() %>%
  drop_na()
fb_minutes_cal <- fb_minutes_cal %>%
  distinct() %>%
  drop_na()
fb_daily_activity <- fb_daily_activity %>%
  distinct() %>%
  drop_na()
fb_hourly_int <- fb_hourly_int %>%
  distinct() %>%
  drop_na()
fb_minutes_int <- fb_minutes_int %>%
  distinct() %>%
  drop_na()
fb_minutes_asleep <- fb_minutes_asleep %>%
  distinct() %>%
  drop_na()
```
*Note: We find there are only 24 unique Id's for the sleep date suggesting that the information may have been lost or was simply not collected for 9 participants in the study.*

As noted previously, we are able to check the validity of some of the data sets by comparing the data gathered and confirming that any previous analysis was performed correctly. We can consolidate the per minute data but using the timetk summarise function as shown
```{r}
# summarize the caloric data by the day
daily_calorie <- fb_minutes_cal %>%
  mutate(ActivityMinute = as.Date(ActivityMinute, format = "%m/%d/%Y") )%>%
  group_by(Id) %>%
  summarise_by_time(.dat_var = ActivityMinute, .by = "day", value = sum(Calories))%>%
  distinct()
head(daily_calorie)

# summarize the intensity data by the hour
hourly_intensity <- fb_minutes_int %>%
    mutate(minutes = strptime(ActivityMinute, "%m/%d/%Y %I:%M:%S %p")) %>%
  group_by(Id) %>%
  summarise_by_time(.dat_var = minutes, .by = "hour", value = sum(Intensity)) %>%
  distinct()
head(hourly_intensity)
```
```{r}
# check for consistency
(n_unique(hourly_intensity$minutes) == n_unique(fb_hourly_int$ActivityHour) && n_unique(hourly_intensity$Id) == n_unique(fb_hourly_int$Id))
(n_unique(daily_calorie$ActivityMinute) == n_unique(fb_daily_activity$ActivityDate) && n_unique(daily_calorie$Id) == n_unique(fb_daily_activity$Id))
```
*Note: The consistency check above lets us know that the manipulated data matches the merged data provided in the spreadsheets.*

As stated previously, each data point in the value column for the sleep data should represent one minute. This can be achieved by simply setting the value to 1 as shown below.
```{r}
# changing all instances of value column to one 
fb_minutes_asleep <- fb_minutes_asleep %>%
  mutate(s_minutes = case_when(fb_minutes_asleep$value != 0 ~ 1 )) 
# summarise the sleep data by the day
daily_sleep <- fb_minutes_asleep %>%
  mutate(date = as.Date(date, format = "%m/%d/%Y") )%>%
  group_by(Id) %>%
  summarise_by_time(.dat_var = date, .by = "day", value = sum(s_minutes))%>%
  rename(minutes_asleep = value) %>%
  distinct()
head(daily_sleep)
# check for consistency
n_unique(daily_sleep$.dat_var) == n_unique(fb_day_asleep$SleepDay)
```
We find that there are incongruities in the date columns for rthe sleep data. The originally merged data set is missing several dates and some instances of the total minutes aslseep are not equivalent per the summarized data above. Lastly, we cannot confidently determine how the total minutes asleep and the total time in bed data columns were differentiated. We are better suited to use the data sets summarized in this review. 

## Analyze and Visualize Data
First we want to look at the activity levels for the participant in the study. The daily activities spreadsheet allows us to examine the daily calories burned which we can then use to categorize the users

```{r}
# remove zeros from the data
cals <- fb_daily_activity[['Calories']]
cals <- cals[cals != 0]
avg_cal <- mean(cals)
std <- sd(cals)
y <- dnorm(cals, avg_cal, std)
```

### Plotting sample & normal distribution curves

A sampling distribution is a probability distribution of a data obtained from a certain number of samples drawn from a specific population. Here we are trying to determine if the sample size collected for this study can be used to predict the outcomes for an entire population (i.e fitbit users). A histogram can be used to visualize the the sample distribution model as below:

```{r}
samp_d <- data.frame(cals,y)
sam_p <- ggplot(samp_d, aes(x=cals))
sam_p <- sam_p + geom_histogram(binwidth = 400, fill = "#69b3a2", color = "black", alpha = 0.9)
sam_p
```

We can then verify the normal distribution by comparing the frequency histogram and the density line like shown below :

```{r}
hist(cals, main = "Frequency plot vs Density line", xlab = "Calories burned per day", ylab = "", yaxt = "n", prob = T ,col = "gray")
lines(density(cals), col = "red", lwd = 2)
```

This information can now be used to define activity level ranges based on the theoretical normal distribution plot, where the levels can be broken into three fitness lifestyles: \* Low \* Moderate \* High

```{r}
lvl_mark <- c((avg_cal+std),(avg_cal+3*std),(avg_cal-std),(avg_cal-3*std))
norm_p <- ggplot(samp_d, aes( x = cals, y = y))
norm_p <- norm_p + geom_line()
norm_p <- norm_p + geom_vline(xintercept = lvl_mark, linetype="dotted", 
                color = "red", size=.5)
norm_p + labs( x = "Calories burned per day", y = "", title = "Normal Distribution Model",)  +
  annotate("text", x = c(avg_cal, (avg_cal+2*std), (avg_cal-2*std)), y = 0.0004, label = c("Moderate", "High", "Low"))+
  theme(
  axis.text.y = element_blank())

```

We can now use these ranges to classify each users activity levels.

```{r}
 
user_avg_cal <- fb_daily_activity %>%
  group_by(Id) %>%
    summarise(avg_cal_burned = mean(Calories))
head(user_avg_cal)
```

```{r}
user_alvl <- user_avg_cal %>%
  mutate (user_alvl = case_when(
    avg_cal_burned <= (avg_cal-std) ~ "Low Physical Activity",
    avg_cal_burned  > (avg_cal-std) & avg_cal_burned<= (avg_cal+std) ~ "Moderate Physical Activity",
    avg_cal_burned > (avg_cal+std) ~ "High Physical Activity"
  ))
  head(user_alvl)
```

Looking at the population stats we can see that an overwhelming percentage of the participants engage in a moderate amount of physical activity.

```{r}
user_demograhics <- user_alvl %>%
  group_by(user_alvl) %>%
  summarise(n_count = n()) %>%
  mutate(total = sum(n_count)) %>%
  group_by(user_alvl) %>%
  summarise(pop_percent = n_count / total) %>%
  mutate(a_lvl = scales::percent(pop_percent)) %>%
  rename(user_activity_level = user_alvl)

user_demograhics %>%
  ggplot(aes(x = "", y=pop_percent, fill=user_activity_level)) +
  geom_bar(stat = "identity", width = 1)+
  coord_polar("y", start=0)+
  theme_minimal()+
  theme(axis.title.x= element_blank(),
        axis.title.y = element_blank(),
        panel.border = element_blank(), 
        panel.grid = element_blank(), 
        axis.ticks = element_blank(),
        axis.text.x = element_blank(),
        plot.title = element_text(hjust = 0.5, size=14, face = "bold")) +
  scale_fill_manual(values = c("darkred", "green", "lightblue")) +
  geom_text(aes(label = a_lvl),
            position = position_stack(vjust = 0.5))+
  labs(title="User Demographics")
```

###View intensity trends throughout the day
Now that we have defined a vague picture the daily user, we decide to paint a more detailed idea of how the users interact with their devices throughtout the day. To make the data easier to consume we want to specify time periods as such.
```{r}
time_of_day <- select(fb_hourly_int, ActivityHour, TotalIntensity) %>%
mutate(hour = ((as.numeric(as.POSIXct(ActivityHour, tz = sys.timezone(), format = "%H:%M"))%%86400)/3600)) %>%
  mutate(time_of_day = case_when(
    hour >= 0 & hour < 5 ~ "Night",
    hour >= 5 & hour < 11 ~ "Morning",
    hour >= 11 & hour < 17 ~ "Daytime",
    hour >= 17 & hour <= 22 ~ "Evening",
    hour == 23 ~ "Night"
  ))
head(time_of_day)
```

We can now group the data by the time of day.

```{r}
tod_intensity <- time_of_day %>%
  group_by(time_of_day) %>%
  summarise(avg_tot_intensity = mean(TotalIntensity))
head(tod_intensity)
```

####Plotting average intensity at different time periods
Lets can take a look at how intensity changes over the course of the month.
```{r}
# find the avg intensity by the hour 
daily_hr_intensity_ <- hourly_intensity %>%
  group_by(minutes) %>%
  summarize_by_time(.date_var = minutes, .by = "hour", value = mean(value))

# plot the avg intensity over time
time_plot <- ggplot(daily_hr_intensity_, aes(x = minutes, y = value)) +
  geom_line(lwd = 0.2)+
  xlab("") +
  ylab("Average Intensity") +
  ggtitle(label = "Average intensity over time")
time_plot
```
We can see that the plot take a somewhat cyclical trend, where it shows similar patterns over the course of a day. We predict that the peak times will occur around the same time each day and we can view this by plotting the average intensity as a function of the time periods we defined previously.
```{r}
ggplot(tod_intensity)+
  coord_flip()+
  geom_col(aes(x = time_of_day, y = avg_tot_intensity),fill = c("lightgreen","green", "forestgreen","darkgreen")) +
  labs (x = "Average intensity", y = "Time of the day", color = "Legend")+
  ggtitle(label = "Avg intensiy throughout a day") 
  
```

*Note that user intensity trends upwards from the early morning hours and peaks in the daytime hours then trend back down as night approaches.*

```{r}
#find the time of peak intensity 
hr_int <- daily_hr_intensity_ %>%
  mutate(hour = hour(minutes)) %>%
  group_by(hour) %>%
  summarize(avg_i_hr = mean(value))

hr_int %>%
  filter_all(any_vars(. %in% max(hr_int$avg_i_hr)))
```

##Understanding sleep patterns
Lastly, we would like to take a look at the relationship between the time spent in bed and the total intensity for a subject on any given day.

```{r distinct users}
# find the each subject avg intensity per day for the 2 month span
avg_daily_intensity <- select(hourly_intensity, Id, minutes, value) %>%
  mutate(m_d_y = as.Date(minutes, format = "%m/%d/%Y")) %>%
  group_by(Id, m_d_y) %>%
  summarise(intensity = mean(value) ) %>%
  rename(date = m_d_y)

# merge the data sets
merged_sleep_int <- merge(avg_daily_intensity, daily_sleep, by = c("Id", "date")) %>%
  select(-c(.dat_var)) 

head(merged_sleep_int)
```

Now we can plot the data to see if there is any correlation in how sleep affects intensity the following day.

```{r observations}
ggplot(merged_sleep_int, aes(x = minutes_asleep, y = intensity)) +
  geom_jitter() +
  geom_smooth() +
  ggtitle("Intensity with respect to sleep time")

intensity <- merged_sleep_int[["intensity"]]
min_sleep <- merged_sleep_int[["minutes_asleep"]]
cor.test(intensity, min_sleep, method = "pearson")
```
The plot and correlation test shows little to no correlation between the two variables. But we can also see that when participants received approx. 6 hours of sleep then they were able to score higher intensity numbers. The optimal amount of sleep that appears to produce the most consistent intensity levels is around 8 hours.


## Interpret & Share Insights
In our analysis, were able to note 3 major findings:

1. 78% of the fitbit users participated in a moderate amount of physical activity daily.
2. The most prolific times of interaction between the user and their devices happened between 11 AM and 11 PM, peaking around 6 PM.
3. Participants are able to exercise more intensely when they average above 6 hours of sleep per night.

Without much information on the participants we are unable to determine a strong marketing reccomendation. However, seeing as most participant engaged in some level of physical activity we can assume cutomers buying products such as this are trying to either improve of maintain some level of fitness appropriate for them. That said, the company should spend more resources in improving the features of the products available or in creating new products to compliment the current offerings. These include ideas such as:

* Enabling users with the ability to track more fitness metrics like weight; blood pressure; or breathing patterns. This can be done through either offering new products or imporving the available devices to collect more user data.
* Using user activity data to optimize/extend battery life through reduced energy consumption during sedentary periods.
* Individualizing the user insight delivered to the consumer by the system application. One suggestion is to provide bedtime recommendations based on energy expenditure throughout the day.
