# 🚴 Cyclistic Case Study 🚴

## Table of Contents
## Ask - Identify research or business task
## Prepare - Acquire data and identify any integrity issues
## Process - Clean data and prepare for analysis
## Analyze - Manipulate data and perform calculations to identify trends and relationships
## Share - Create data visualizations
## Act - Share conclusions and offer data-driven insights

## Ask
Cyclistic is a fictional bike-share company looking to tailor their marketing strategy to entice more 'casual' riders (i.e. riders that purchase single-ride or one-day passes) into annual members. The business task is to figure out how casual riders and annual members differ in their behavior so as to identify how Cyclistic could better target how to make membership more appealing to casual riders.

## Prepare
Cyclistic data used for this project is located at [https://divvy-tripdata.s3.amazonaws.com/index.html](https://divvy-tripdata.s3.amazonaws.com/index.html). I used each month's data for 2023. Due to the size of the datasets, I'll be using R to process the data. I started by combining the individual CSV files into a single dataframe. From there, I noted some initial limitations of the data:

- The size of the combined datasets exceeded limitations of Excel and Google Sheets, so could not be (easily) imported into a single spreadsheet.
- Due to user data-privacy limitations, riders' personally identifiable information is not available. So, for example, riders cannot be connected to ride IDs, indicating whether one rider may be a frequent 'casual' rider but not a member.
- The data contained redundant information for the purposes of this analysis (e.g. station IDs).

## Process

### Transformation
I started with adding a column titled 'ride_length' that is the result of subtracting the values of column 'started_at' from 'ended_at'. I then rounded the resulting values to the nearest whole minute.

```
#Create ride_length column
data_2023 <- data_2023 |>
  mutate(ride_length = difftime(ended_at, started_at, units = "mins"))

#Round ride_length column to nearest whole minute
data_2023$ride_length <- round(data_2023$ride_length, 0)
```
I then added columns 'start_date', 'end_date', 'start_time', 'end_time', 'day_of_week', and 'month', using values from columns 'started_at' and 'ended_at', respectively.

```
#Create start_date column
data_2023$start_date <- as.Date(data_2023$started_at)

#Create end_date column
data_2023$end_date <- as.Date(data_2023$ended_at)

#Create start_time column
data_2023$start_time <- format(as.POSIXct(data_2023$started_at), format = "%H:%M:%S")

#Create end_time column
data_2023$end_time <- format(as.POSIXct(data_2023$ended_at), format = "%H:%M:%S")

#Create day_of_week column using lubridate package
data_2023$day_of_week <- wday(data_2023$started_at, label = TRUE)

#Create month column
data_2023$month <- month(data_2023$started_at, label = TRUE)
```

Next, I changed the names of column 'member_casual' to 'rider_type', and 'rideable_type' to 'bike_type'.

```
data_2023 <- data_2023 |>
  rename(bike_type = rideable_type)

data_2023 <- data_2023 |>
  rename(rider_type = member_casual)
```

Lastly, I removed columns irrelevant for analysis in this case study. For example, the values in column 'ride_id' cannot be connected to specific customers due to data-privacy issues.

```
data_2023 <- select(data_2023, -c(ride_id, start_station_id, end_station_id, start_lat, start_lng, end_lat, end_lng))
```


### Cleaning
I then began some basic cleaning of the data: removing NA values, duplicate rows, and rows that included a value less than zero in the 'ride_length' column (as having a negative trip duration is impossible).

```
#Remove any rows with NA values
data_2023 <- na.omit(data_2023)

#Remove any duplicate rows
data_2023 <- unique(data_2023)

#Remove any rows where the value in 'ride_length' is less than 0
data_2023 <- data_2023[data_2023$ride_length >= 0, ]
```

Next, I removed some rows meeting certain criteria which represented potentially unreliable data. This was assigned to a new dataframe 'data_2023_filtered'. The following rows were filtered using the subset function.
1. Rows where 'bike_type' was 'docked_bike'.
2. Rows where 'ride_length' was less than 1 minute.
3. Rows where 'ride_length' exceeded 600 minutes (10 hours). (I will further filter the data in the analysis process to make visualization easier)

```
#Filter rows where 'bike_type' equals 'docked_bike'
data_2023_filtered <- subset(data_2023, bike_type != 'docked_bike')

#Filter rows where 'ride_length' is less than 1 minute or greater than 600 minutes.
data_2023_filtered <- subset(data_2023, ride_length > 0 & ride_length <= 600)
```

## Analyze

First, I calculated the mean and standard deviation of the 'ride_length' column. This will allow me to identify outliers that could skew visualizations.

```
#Calculate the mean of 'ride_length'
ride_length_mean <- mean(data_2023_filtered$ride_length)

#Calculate the standard deviation of 'ride_length'
ride_length_sd <- sd(data_2023_filtered$ride_length)
```
The result of ride_length_mean is 14.39897 mins and the result of ride_length_sd is 19.20853. The two sigma threshold would therefore be 52.81603, representative of 95% of the (filtered) dataset. Since the column 'ride_length' deals with duration, there is no need to calculate the lower threshold, as it would be a negative duration.

From this result, I created a new dataframe 'data_2023_filtered_sd' which excludes rows that contain a value in 'ride_length' greater than 53 mins. This dataframe will be used to create visualizations involving ride length.

```
data_2023_filtered_sd <- subset(data_2023_filtered, ride_length <= 53)
```

Then, I did some basic calculations on the data to highlight differences between casual riders and members.

```
#Calculate mean ride length for casual riders
casual_rider_mean_length <- data_2023_filtered |>
  filter(rider_type == "casual") |>
  summarise(casual_rider_mean_length = mean(ride_length))
#Print the result
print(casual_rider_mean_length)

#Calculate mean ride length for members
member_rider_mean_length <- data_2023_filtered |>
  filter(rider_type == "member") |>
  summarise(member_rider_mean_length = mean(ride_length))
#Print the result
print(member_rider_mean_length)

#Calculate the mode for the column 'day_of_week' using DescTools mode() function
day_of_week_mode <- Mode(data_2023_filtered$day_of_week)
#Print the result
print(day_of_week_mode)
```
I then prepared the data for visualization while analyzing the data. First, I created two new dataframes comparing rider type behaviors: one that groups number of riders per month by rider type and one that groups numbers of riders per day by rider type.

```
#Create new dataframe that groups numbers of riders per month by rider type
ride_count_by_month <- data_2023_filtered |>
  group_by(month, rider_type) |>
  summarise(ride_count = n())

#Create new dataframe that groups numbers of riders per day by rider type
ride_count_by_day <- data_2023_filtered |>
  group_by(day_of_week, rider_type) |>
  summarise(ride_count = n())
```

## Share

I created the following visualizations to determine relationships between casual riders and members.

### Bar Graph of Monthly Rides by Rider Type

```
ggplot(ride_count_by_month, aes(x = month, y = ride_count, group = rider_type, fill = rider_type)) +
geom_bar(stat = "identity", position = "dodge") +
facet_wrap(~ month, scales = 'free_x') +
labs(x = "Month",
y = "Number of Rides",
title = "Monthly Rides by Rider Type") +
theme_minimal() +
scale_y_continuous(labels = label_number())
```

![Monthly Rides By Rider Type Bar Graph](https://github.com/griffindeutsch/google_data_analytics_capstone/assets/63735165/092a9ac2-adfc-471c-bbe8-5ddc17b07696)

## Bar Graph of Daily Rides by Rider Type

```
ggplot(ride_count_by_day, aes(x = day_of_week, y = ride_count, group = rider_type, fill = rider_type)) +
geom_bar(stat = "identity", position = "dodge") +
facet_wrap(~ day_of_week, scales = 'free_x') +
labs(x = "Day",
y = "Number of Rides",
title = "Daily Rides by Rider Type") +
theme_minimal() +
scale_y_continuous(labels = label_number())
```

![Daily Rides by Rider Type Bar Graph](https://github.com/griffindeutsch/google_data_analytics_capstone/assets/63735165/7373b8ca-488f-4daf-93f2-6eae2b81895f)

### Histogram of Ride Length by Rider Type
```
ggplot(data_2023_filtered_sd, aes(x = ride_length, fill = rider_type)) +
  geom_histogram(position = "dodge", binwidth = 1) +
    labs(title = "Ride Length by Rider Type",
    x = "Ride Length (minutes)",
    y = "Number of Rides") +
  theme_minimal() +
  scale_y_continuous(labels = label_number())
```
We can see from this histogram that while both members and casual riders ride around the same duration on average, many more short trips are by members rather than casual riders.

![Ride Length by Rider Type Histogram](https://github.com/griffindeutsch/google_data_analytics_capstone/assets/63735165/a885bd2c-08ad-45fa-9a93-3b22f3b6396c)

### Horizontal Bar Graphs of Average Ride Lengths by Rider Type

I then wanted to compare trip duration between members and casual riders across months and days. I created two separate horizontal bar graphs to do this.

```
#Create horizontal bar graph showing Average Ride Length by Rider Type by Month
data_2023_filtered_sd |>
  group_by(month, rider_type) |>
  summarise(average_ride_length = mean(ride_length)) |>
    ggplot(aes(x = month, y = average_ride_length, fill = rider_type, label = round(average_ride_length, 1))) +
    geom_bar(stat = "identity", position = "dodge") +
    geom_text(aes(y = round(average_ride_length, 1))) +
    coord_flip() +
      labs(title = "Average Ride Length by Rider Type (by Month)",
          x = "Month",
          y = "Average Ride Length (mins)") +
    theme_minimal()

#Create horizontal bar graph showing Average Ride Length by Rider Type by Day of Week
data_2023_filtered_sd |>
  group_by(day_of_week, rider_type) |>
  summarise(average_ride_length = mean(ride_length)) |>
    ggplot(aes(x = day_of_week, y = average_ride_length, fill = rider_type, label = round(average_ride_length, 1))) +
    geom_bar(stat = "identity", position = "dodge") +
    geom_text(aes(y = round(average_ride_length, 1))) +
    coord_flip() +
      labs(title = "Average Ride Length by Rider Type (by Day)",
          x = "Day of Week",
          y = "Average Ride Length (mins)") +
    theme_minimal()
```

Average ride length was 2.33 minutes longer on average per month for casual riders.

![Average Ride Length by Rider Type - Month](https://github.com/griffindeutsch/google_data_analytics_capstone/assets/63735165/a1953667-6ded-4d65-93bd-b38bad77b521)

Average ride length was 2.82 minutes longer on average per day for casual riders.

![Average Ride Length by Rider Type - Day](https://github.com/griffindeutsch/google_data_analytics_capstone/assets/63735165/c08bcd92-0242-49b9-ad2d-657292aa5760)


## Bar Graph of Monthly Rides by Bike Type

```
ggplot(ride_count_by_bike_type, aes(x = month, y = ride_count, group = bike_type, fill = bike_type)) +
geom_bar(stat = "identity", position = "dodge") +
facet_wrap(~ month, scales = 'free_x') +
labs(x = "Month",
y = "Number of Rides",
title = "Number of Monthly Rides by Bike Type") +
theme_minimal() +
scale_y_continuous(labels = label_number())
```

![Number of Monthly Rides by Bike Type Bar Graph](https://github.com/griffindeutsch/google_data_analytics_capstone/assets/63735165/f9abac48-f147-4472-86f5-3160687f1c12)


## Bar Graph of Daily Rides by Bike Type

```
ggplot(ride_count_by_bike_type, aes(x = day_of_week, y = ride_count, group = bike_type, fill = bike_type)) +
geom_bar(stat = "identity", position = "dodge") +
facet_wrap(~ day_of_week, scales = 'free_x') +
labs(x = "Day",
y = "Number of Rides",
title = "Number of Daily Rides by Bike Type") +
theme_minimal() +
scale_y_continuous(labels = label_number())
```

![Number of Daily Rides by Bike Type Bar Graph](https://github.com/griffindeutsch/google_data_analytics_capstone/assets/63735165/3d924154-1312-4bc8-80d0-236e7f932013)

## Stacked Bar Graph of Bike Type by Rider Type

```
ggplot(bike_type_by_rider_type, aes(x = bike_type, y = ride_count, fill = rider_type)) +
geom_bar(stat = "identity", position = "stack") +
labs(x = "Bike Type",
y = "Number of Rides",
title = "Bike Type Usage by Rider Type") +
theme_minimal() +
scale_y_continuous(labels = label_number())
```
![Bike Type Usage by Rider Type](https://github.com/griffindeutsch/google_data_analytics_capstone/assets/63735165/61b83e4e-1383-4359-a611-c550c8b5f726)


## Act

From my analysis, we can observe the following:

1. There are about 150,000 more rides by members than causal riders per month across each month, with ridership for both categories being highest in summer and lowest in winter.
2. Differences in daily number of rides between the two categories of rider involves more variation across days of the week, with Wednesday showing the largest difference between members and casual riders (about 425,000 more member rides on Wednesdays than casual rides). Saturdays and Sundays showed the smallest difference in number of rides between rider types with approximately 100,000 more member rides than casual rides.
3. Member and casual rides show a similar frequency distribution in ride length relative to their overall number of rides. Most rides of about 5 minutes are taken by members, while rides exceeding 40 minutes in length were more frequently taken by casual riders.
4. Casual riders consistenly ride for longer than members.
5. Electric bikes are rode more often than classic bikes in every month except August and September.
6. Electric bikes are rode more often than classic bikes throughout each day of the week.
7. Members ride classic bikes and electric bikes at essentially the same rate, while casual riders ride electric bikes more frequently than classic bikes.

From these observations, I drew the following conclusions as to how Cyclistic could convert more casual riders to annual members:

1. **Further incentivize electric bikes in the membership program** - electric bikes are ridden more frequently overall by casual riders, perhaps showing that shorter ride length is preferable to casual riders to keep the price of the trip low. Alternatively, the price difference between classic and electric bikes may not be large enough to justify not using the electric bike for one-off rides.
2. **Further incentivize riding during weekdays** - members ride much more frequently than casual riders during weekdays, highlighting the use of bikes for commuting. If a commuter pass was introduced that was priced similar to passes for other forms of transportation, casual riders may be more likely to ride to work during the week.
3. **Adjust pricing for membership program to account for longer trips** - although casual riders ride electric bikes more frequently than classic bikes, they have consistently longer trips than members. This potentially indicates the price point for the intital undocking for casual riders being too high to justify shorter rides. Casual riders may not be saving enough in undocking fees with the membership to account for the extra upfront cost of membership.
4. **Offer additional membership benefits during winter months** - ridership across both rider types is significantly lower in winter months. If Cyclistic offered an additional benefit during these months, such as no undocking fee for either bike type (as bike type usage is roughly equivalent during these months), it may attract casual riders to consider an electric bike, for example, as a primary mode of transport.


