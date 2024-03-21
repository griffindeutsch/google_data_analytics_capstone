# 🚴 Cyclistic Case Study 🚴

## Ask - Identify research/buisness task
## Prepare - Acquire data and identify any integrity issues
## Process - Clean data and prepare for analysis
## Analyze - Manipulate data and perform calculations to identify trends and relationships
## Share -  Create data visualizations
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
3. Rows where 'ride_length' exceeded 600 minutes (10 hours).

```
#Filter rows where 'bike_type' equals 'docked_bike'
data_2023_filtered <- subset(data_2023, bike_type != 'docked_bike')

#Filter rows where 'ride_length' is less than 1 minute or greater than 600 minutes.
data_2023_filtered <- subset(data_2023, ride_length > 0 & ride_length <= 600)
```

## Analyze

First, I did some basic calculations on the data to highlight differences between casual riders and members.

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
I then prepared the data for visualization while analyzing the data. First, I created two new dataframes: one that groups number of riders per month by rider type and one that groups numbers of riders per day by rider type.

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





## Act











