# Google Data Analytics: Chicago BikeShare Capstone Project
***Author***: Ali Shalaby

***Date***: July 22, 2022

:art: [Dashboard Viz](https://public.tableau.com/app/profile/ali.shalaby/viz/BikeShareDashboard_16583710588640/Dashboard1)

## Overview
- Analyzed Cyclistic's bikeshare riding data to develop marketing strategies in order to ***convert casual users to members***
- Retrieved, grasped and organized data structure to a suitable format for analysis
- ***Cleaned*** and ***analyzed data*** using R's dplyR package
- ***Vizualized*** queries through R's ggplot2 package
- Transformed visualizations to ***KPI dashboard*** to present findings
- Developed marketing strategies based on findings

## 1. Ask

***BUSINESS TASK***: Design marketing strategies aimed at converting casual riders into annual members.
- How do annual members and casual riders use Cyclistic's bikeshare differently?
- Why would casual riders buy Cyclistic annual memberships?
- How can Cyclistic use digital media to influence casual riders to become members?

## 2. Prepare

***Download and store dataset*** 
<br />[Dataset](https://divvy-tripdata.s3.amazonaws.com/index.html): dataset used  contained 12 csv files indexed by year (August 2020-August 2021). 
<br /> The data has been made available by Motivate International Inc.

***Identify how Data is Organized***
<br /> All trip data is in CSV format with 13 columns, including: ride ID #, ride type, start/end time, start/end station name and id, long and lat, and member/casual status

***Determine Data Credibility and Integrity***
<br /> First party data that follows ROCC approach:
- ***Reliable***: the data includes complete and accurate ride data from Divvy; a program of the Chicago Department 
<br /> of Transportation (CDOT), which owns the city’s bikes, stations and vehicles
- ***Original***: the data is from Motivate International Inc, which operates the City of Chicago’s Divvy bicycle sharing service.
- ***Comprehensive***: The data incudes type of bikes, start and end station name, start and end time, station ID, station longtitude and latitude, membership types.
- ***Current***: data is up to date to August 2021
- ***Cited***: the data is cited and under current [license agreement](https://ride.divvybikes.com/data-license-agreement).

## 3. Process

***Check Data Types and Structure***

```
head(bike_data)
str(bike_data)
summary(bike_data)
```

***Check for missing values***
<br /> The code chunk below checks both total number of NA values in the table and per column. Evident that most na values are contained in station-based columns: 1. start_station_name 2. start_station_id 3. end_station_name 4. end_station_id We should leave the data as is and not replace NA values
```
sum(is.na(bike_data)) 
bike_data %>% summarise_all(~ sum(is.na(.))) 
```

***Remove unwanted data***
```
bike_data <- bike_data %>% 
  select(-c(start_lat, start_lng, end_lat, end_lng))
```

***Transform data for more effective analysis***
<br /> The code chunk below adds variables 'weekday', 'ride_length', and 'month'
```
bike_data <- 
  bike_data %>% 
  mutate(weekday = weekdays(as.Date(bike_data$started_at))) %>%
  mutate(ride_length = ended_at - started_at) %>% 
  mutate(month = months(as.Date(bike_data$started_at)))
```

***Filter out bad data***
<br /> The code chunk below removes observations where bikes were used for less than a minute and more than a day
```
bike_data <- bike_data %>%
  filter(ride_length >1) %>% 
  filter(ride_length <= 1440) 
```
## 4. Analyze

***Aggregate data based on casual vs member riders***
```
bike_data %>% 
  group_by(member_casual) %>% 
  summarise(number_of_riders = n(), 
            mean = mean(ride_length),
            median = median(ride_length))
```
<img src="https://user-images.githubusercontent.com/83675013/180481258-ed8ace66-678e-4bfd-aa7e-5b2e4ff473e4.jpeg" width="500" height="300" />

***Analyze average ride length by user type and day of the week***
```
rider_weekday <- bike_data %>% 
  group_by(member_casual, weekday) %>% 
  summarise(average_ride_length = mean(ride_length), number_of_rides = n())
  
rider_weekday$weekday <- factor(rider_weekday$weekday, levels= c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))
rider_weekday <- rider_weekday[order(rider_weekday$weekday),]

rider_weekday %>% 
  ggplot(aes(x=weekday, y=average_ride_length, fill=member_casual)) + 
  geom_col() +
  labs(title="Average Weekly Ride Length", x="Week", y="Average Ride Length", fill="User Type") + 
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1))
```
<img src="https://user-images.githubusercontent.com/83675013/180483749-75a02428-dd94-4f79-9834-3dd169d0c550.jpeg" width="500" height="300" />

:rotating_light: For complete analysis and visualization code chunks, view the [rmd file](https://github.com/alishalaby07/Chicago-BikeShare-Analysis-Capstone/blob/main/BikeShareAnalysis.Rmd) attached here.


