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
I then added columns 'start_date', 'end_date', 'start_time', and 'end_time', using values from columns 'started_at' and 'ended_at', respectively.

```
#Create start_date column
data_2023$start_date <- as.Date(data_2023$started_at)

#Create end_date column
data_2023$end_date <- as.Date(data_2023$ended_at)

#Create start_time column
data_2023$start_time <- format(as.POSIXct(data_2023$started_at), format = "%H:%M:%S")

#Create end_time column
data_2023$end_time <- format(as.POSIXct(data_2023$ended_at), format = "%H:%M:%S")
```
Lastly, I removed columns irrelevant for analysis in this case study. For example, the values in column 'ride_id' cannot be connected to specific customers due to data-privacy issues, and the columns containing coordinates are redundant as station start and end names are also given.

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


## Analyze

## Share

## Act











