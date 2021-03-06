Bikesharing Case Study
================

-   [Problem Statement](#problem-statement)
-   [1. Packages and libraries](#1-packages-and-libraries)
-   [2. Prepare](#2-prepare)
    -   [2.1 Import data](#21-import-data)
    -   [2.2 Checking data structure](#22-checking-data-structure)
    -   [2.3 Merge datasets](#23-merge-datasets)
-   [3. Process](#3-process)
-   [4. Analyze](#4-analyze)
    -   [4.1 Descriptive analysis](#41-descriptive-analysis)
    -   [4.2 Most popular bike stations](#42-most-popular-bike-stations)
    -   [4.3 Visualizing the number of
        trips](#43-visualizing-the-number-of-trips)
        -   [4.3.1 By month](#431-by-month)
        -   [4.3.2 By weekday](#432-by-weekday)
        -   [4.3.3 By time of day](#433-by-time-of-day)
    -   [4.4 Trip length](#44-trip-length)
    -   [4.5 Bike type preference](#45-bike-type-preference)
-   [5. Recommendations](#5-recommendations)
-   [Next steps](#next-steps)

------------------------------------------------------------------------

<br>

## Problem Statement

This case study is part of the Google Data Analyst Certificate. I will
be looking at the fictional “Cyclistic” company and try to gain
important insights from historical bike trip data.

In general a bike-sharing system is a service that makes bikes available
for shared use to individuals on a short term basis in exchange for a
certain fee. Usually people can borrow a bike from different
computer-controlled docks (also called stations). Once they enter their
payment information the system unlocks the bike so that it’s free to
use. These bikes can be returned to every docking station belonging to
the same provider.

So far *Cyclistic’s* marketing strategy has relied on building general
awareness and appealing to broad consumer segments. They categorize
their costumers into **casual**-users (people who purchase single-ride
or full-day passes) and **member**-users (customers who purchase an
annual membership).

Cyclistic’s ﬁnance analysts have concluded that annual members are much
more proﬁtable than casual riders. Hence, the company believes that
maximizing the number of annual members will be key to future growth.
According to the marketing team there is a good chance to convert casual
riders into members. In order to do that, it is important to identify
how annual members and casual riders use the service differently and
come up with possibilities to incentives casual riders to go for a
membership option.

<br>

## 1. Packages and libraries

The code below ensures that all required packages are installed and
loaded.

``` r
# For data import, data wrangling and visualization (ggplot2)
if(!require(tidyverse)){
  install.packages("tidyverse")
}
library(tidyverse)

# For date functions
if(!require(lubridate)){
  install.packages("lubridate")
}
library(lubridate)

# For time data
if(!require(chron)){
  install.packages("chron")
}
library(chron)

# For Export of this documentation
if(!require(rmarkdown)){
  install.packages("rmarkdown")
}
library(rmarkdown)

if(!require(knitr)){
  install.packages("knitr") 
}
library(knitr)

# For mapping
if(!require(ggmap)){
  install.packages("ggmap") 
}
library(ggmap)

# For percentages
if(!require(formattable)){
  install.packages("formattable") 
}
library(formattable)

# For arranging plots
if(!require(cowplot)){
  install.packages("cowplot") 
}
library(cowplot)
```

<br>

## 2. Prepare

Bear in mind that *Cyclistic* is a fictional company. The raw data has
been collected by *Motivate International Inc* (can be downloaded [here](https://divvy-tripdata.s3.amazonaws.com/index.html)). This is the
company which operates the City of Chicago’s Divvy bicycle sharing
service. The license to use this public dataset can be found
[here](https://ride.divvybikes.com/data-license-agreement).

The data has already been pre-processed by Divvy to remove trips that
were taken by staff as they service and inspect the system and any trips
that were below 60 seconds in length (potentially false starts or users
trying to re-dock a bike). Each month has around 100.000 - 900.000
entries of data saved in a separate csv file with the same headings. Due
to the large file sizes, R has been used to clean and process the
datasets.

To ensure data privacy each trip has been anonymized and only includes
information such as ride_id (serving as the table’s primary key to
identify each bike trip), the type of bike used, details about the
starting and ending docking station (including name, ID, latitude and
longitude), the type of customer (casual or member) and the date and
time for when the bike was picked up and dropped off.

I will be examining a period of 12 months, starting from January 2021 to
the end of december 2021.

### 2.1 Import data

The code below reads in the csv-files of each month of 2021 into a data
frame.

``` r
bikedata_jan_2021 <- read.csv("2021-01-tripdata.csv") # January 2021

bikedata_feb_2021 <- read.csv("2021-02-tripdata.csv") # February 2021 

bikedata_mar_2021 <- read.csv("2021-03-tripdata.csv") # March 2021

bikedata_apr_2021 <- read.csv("2021-04-tripdata.csv") # April 2021

bikedata_may_2021 <- read.csv("2021-05-tripdata.csv") # May 2021

bikedata_jun_2021 <- read.csv("2021-06-tripdata.csv") # June 2021

bikedata_jul_2021 <- read.csv("2021-07-tripdata.csv") # July 2021

bikedata_aug_2021 <- read.csv("2021-08-tripdata.csv") # August 2021

bikedata_sep_2021 <- read.csv("2021-09-tripdata.csv") # September 2021

bikedata_oct_2021 <- read.csv("2021-10-tripdata.csv") # October 2021

bikedata_nov_2021 <- read.csv("2021-11-tripdata.csv") # November 2021

bikedata_dec_2021 <- read.csv("2021-12-tripdata.csv") # December 2021
```

### 2.2 Checking data structure

The *str()* command helps to identify if any of the individual raw
datasets have different column names, data types etc.

``` r
str(bikedata_jan_2021)
str(bikedata_feb_2021)
str(bikedata_mar_2021)
str(bikedata_apr_2021)
str(bikedata_may_2021)
str(bikedata_jun_2021)
str(bikedata_jul_2021)
str(bikedata_aug_2021)
str(bikedata_sep_2021)
str(bikedata_oct_2021)
str(bikedata_nov_2021)
str(bikedata_dec_2021)
```

### 2.3 Merge datasets

Since every column has the same name and there are no datatype
anomalies, the 12 individual datasets are ready to be merged into one
full-year dataframe containing all ride data from 2021. To relieve the main memory, the individual datasets are removed.

``` r
# Merge all 12 individual datasets into one full-year datafame
biketrips2021 <- rbind(bikedata_jan_2021, bikedata_feb_2021, bikedata_mar_2021, bikedata_apr_2021, bikedata_may_2021, bikedata_jun_2021, bikedata_jul_2021, bikedata_aug_2021, bikedata_sep_2021, bikedata_oct_2021, bikedata_nov_2021, bikedata_dec_2021)

# Remove individual datasets from main memory
rm(bikedata_jan_2021, bikedata_feb_2021, bikedata_mar_2021, bikedata_apr_2021, bikedata_may_2021, bikedata_jun_2021, bikedata_jul_2021, bikedata_aug_2021, bikedata_sep_2021, bikedata_oct_2021, bikedata_nov_2021, bikedata_dec_2021)
```

<br>

## 3. Process

Next up, the data has to be checked for errors and transformed in a way
that one can work effectively with it. This cleaning and tidying process
will be documented in the following.

To get a short preview of the data the *head()* function is very useful.
This allows me to see if any irrelevant columns need to be omitted.
Furthermore, it is once again useful to work with the *str()* function
to inspect the data and look for any abnormalties.

``` r
head(biketrips2021)
```

    ##            ride_id rideable_type          started_at            ended_at
    ## 1 E19E6F1B8D4C42ED electric_bike 2021-01-23 16:14:19 2021-01-23 16:24:44
    ## 2 DC88F20C2C55F27F electric_bike 2021-01-27 18:43:08 2021-01-27 18:47:12
    ## 3 EC45C94683FE3F27 electric_bike 2021-01-21 22:35:54 2021-01-21 22:37:14
    ## 4 4FA453A75AE377DB electric_bike 2021-01-07 13:31:13 2021-01-07 13:42:55
    ## 5 BE5E8EB4E7263A0B electric_bike 2021-01-23 02:24:02 2021-01-23 02:24:45
    ## 6 5D8969F88C773979 electric_bike 2021-01-09 14:24:07 2021-01-09 15:17:54
    ##           start_station_name start_station_id end_station_name end_station_id
    ## 1 California Ave & Cortez St            17660                                
    ## 2 California Ave & Cortez St            17660                                
    ## 3 California Ave & Cortez St            17660                                
    ## 4 California Ave & Cortez St            17660                                
    ## 5 California Ave & Cortez St            17660                                
    ## 6 California Ave & Cortez St            17660                                
    ##   start_lat start_lng end_lat end_lng member_casual
    ## 1  41.90034 -87.69674   41.89  -87.72        member
    ## 2  41.90033 -87.69671   41.90  -87.69        member
    ## 3  41.90031 -87.69664   41.90  -87.70        member
    ## 4  41.90040 -87.69666   41.92  -87.69        member
    ## 5  41.90033 -87.69670   41.90  -87.70        casual
    ## 6  41.90041 -87.69676   41.94  -87.71        casual

``` r
str(biketrips2021)
```

    ## 'data.frame':    5595063 obs. of  13 variables:
    ##  $ ride_id           : chr  "E19E6F1B8D4C42ED" "DC88F20C2C55F27F" "EC45C94683FE3F27" "4FA453A75AE377DB" ...
    ##  $ rideable_type     : chr  "electric_bike" "electric_bike" "electric_bike" "electric_bike" ...
    ##  $ started_at        : chr  "2021-01-23 16:14:19" "2021-01-27 18:43:08" "2021-01-21 22:35:54" "2021-01-07 13:31:13" ...
    ##  $ ended_at          : chr  "2021-01-23 16:24:44" "2021-01-27 18:47:12" "2021-01-21 22:37:14" "2021-01-07 13:42:55" ...
    ##  $ start_station_name: chr  "California Ave & Cortez St" "California Ave & Cortez St" "California Ave & Cortez St" "California Ave & Cortez St" ...
    ##  $ start_station_id  : chr  "17660" "17660" "17660" "17660" ...
    ##  $ end_station_name  : chr  "" "" "" "" ...
    ##  $ end_station_id    : chr  "" "" "" "" ...
    ##  $ start_lat         : num  41.9 41.9 41.9 41.9 41.9 ...
    ##  $ start_lng         : num  -87.7 -87.7 -87.7 -87.7 -87.7 ...
    ##  $ end_lat           : num  41.9 41.9 41.9 41.9 41.9 ...
    ##  $ end_lng           : num  -87.7 -87.7 -87.7 -87.7 -87.7 ...
    ##  $ member_casual     : chr  "member" "member" "member" "member" ...

All of these columns are relevant for the analysis, so I’ll keep them.
Besides, we can see the data in columns `started_at` and `ended_at` are
saved as a character string. In order to allow for calculations these
columns have to be converted into a time data type. The *strptime*
command helps to convert a character string into a time data type.

``` r
# Change data type of columuns 'started_at' and 'ended_at'
biketrips2021$started_at <- strptime(biketrips2021$started_at,"%Y-%m-%d %H:%M:%S")
biketrips2021$ended_at <- strptime(biketrips2021$ended_at,"%Y-%m-%d %H:%M:%S")

# Sort by date
biketrips2021 <- arrange(biketrips2021, started_at)
```

Now the data is ready for further cleaning and tidying steps. I will
create a new version of the dataframe (v2) since data is being removed.

``` r
biketrips2021_v2 <- biketrips2021 %>%
# Rename column 'member_casual' to 'usertype'
  rename(usertype = member_casual) %>%
  
# Create a new column that computes each ride's duration
  mutate(duration = ended_at - started_at) %>%

# Remove all trips that were below 60 seconds in length (potentially false starts or users trying to re-dock a bike) and outliers of people using bikes for longer than 48 hours (potentially forgetting or failing to end the service)
  filter(duration >= 60 & duration < 172800) %>% 
  
# Remove incomplete rows with NA or blank station names
  filter(!(is.na(start_station_name) | start_station_name == "")) %>% 
  filter(!(is.na(end_station_name) | end_station_name == ""))
```

The ride_id column is unique to each ride. This column should be
reviewed to see if there are any duplicates to delete.

``` r
# Create a data frame to check if are duplicates 
duplicates <- biketrips2021_v2 %>%
  count(ride_id) %>%
  filter(n > 1)
duplicates
```

    ## [1] ride_id n      
    ## <0 Zeilen> (oder row.names mit Länge 0)

Since this code returns no results, the dataset does not contain
duplicate rows.

<br>

## 4. Analyze

The data had been stored, prepared and processed, hence it’s time to
start putting it to work. Key tasks are to aggregate the data to ensure
it is useful and accessible, to organize and format it in order to
perform calculations and finally identify trends and relationships.

First, I will add columns that list the *date*, *year*, *month*, *week*,
*day* and *time of day* of each ride. This allows to aggregate the data
for example for each day, month or year. Before that, the data could
only be aggregated at the ride level.

``` r
# Date (YYYY-MM-DD)
biketrips2021_v2$date <- as.Date(biketrips2021_v2$started_at)

# Year
biketrips2021_v2$year <- as.numeric(format(biketrips2021_v2$started_at, "%Y"))

# Month
biketrips2021_v2$month <- format(biketrips2021_v2$started_at, "%m")
   # Replace string numbers with actual names
biketrips2021_v2$month <- str_replace_all(biketrips2021_v2$month, c( "01"="Jan", "02"="Feb", "03"="Mar", "04"="Apr", "05"="May", "06"="Jun", "07"="Jul", "08"="Aug", "09"="Sep", "10"="Oct", "11"="Nov", "12"="Dec"))
   # Arrange months in order
biketrips2021_v2$month <- ordered(biketrips2021_v2$month, levels = c("Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"))

# Week 
biketrips2021_v2$week <- as.numeric(format(biketrips2021_v2$started_at, "%W"))

# Day
biketrips2021_v2$day <- as.numeric(format(biketrips2021_v2$started_at, "%d"))

# Weekday 
biketrips2021_v2$weekday <- format(biketrips2021_v2$started_at, "%A")
  # Arrange weekdays in order
biketrips2021_v2$weekday <- ordered(biketrips2021_v2$weekday, levels = c("Montag", "Dienstag", "Mittwoch", "Donnerstag", "Freitag", "Samstag", "Sonntag"))

# Time of Day
biketrips2021_v2$time_of_day <- format(biketrips2021_v2$started_at, "%H:%M:%S")
```

<br>

### 4.1 Descriptive analysis

Next, I will conduct some simple descriptive analysis on the ride
*duration*.

``` r
# Create a column with all ride durations in minutes
biketrips2021_v2 <- mutate(biketrips2021_v2, duration_min = duration/60)
biketrips2021_v2$duration_min <- as.numeric(biketrips2021_v2$duration_min)

mean(biketrips2021_v2$duration_min)   # average duration of all rides
```

    ## [1] 20.38812

``` r
median(biketrips2021_v2$duration_min) # mean duration of all ride
```

    ## [1] 12.36667

``` r
# Compare members and casual users
usertype_comparison <- biketrips2021_v2 %>%
  group_by(usertype) %>%
  summarize(avg_ride_min = mean(duration_min), 
            median_ride_min = median(duration_min), longest_ride_min = 
            max(duration_min), number_of_rides = n()) %>%
  mutate(longest_ride_hour = longest_ride_min/60) %>% 
  mutate_if(is.numeric, round, digits = 2)
usertype_comparison
```

    ## # A tibble: 2 × 6
    ##   usertype avg_ride_min median_ride_min longest_ride_min number_of_rides
    ##   <chr>           <dbl>           <dbl>            <dbl>           <dbl>
    ## 1 casual           29.0           16.8             2868.         2027165
    ## 2 member           13.4            9.87            1496.         2500962
    ## # … with 1 more variable: longest_ride_hour <dbl>

**Key characteristics**

All users:

-   Average ride duration: 20.39 minutes  
-   Median ride duration: 12.37 minutes

Only casual riders:

-   Average ride duration: 29.03 minutes  
-   Median ride duration: 16.83 minutes  
-   Longest ride (\< 48 hours): 47.81 hours

Only members:

-   Average ride duration: 13.38 minutes
-   Median ride duration: 9.87 minutes
-   Longest ride (\< 48 hours): 24.93 hours  
    <br>

### 4.2 Most popular bike stations

To find out which stations are most frequently used by each usertype,
every station will be mapped on a topographical depiction of Chicago and
highlighted by color according to the number of rides.

``` r
# Create a data frame which groups number of trips by station name and usertype while containing each stations longitude and latitude information
mapping_data <- biketrips2021_v2 %>%
  select(start_station_name, start_lat, start_lng, usertype) %>%
  group_by(start_station_name,usertype) %>%
  mutate(numtrips = n()) %>%
  distinct(start_station_name,  .keep_all = TRUE) %>%
  arrange(numtrips)
# Define box
sbbox <- make_bbox(lon = c(-87.9, -87.5), lat = c(42.1, 41.65), f = .1)
# Get map
chicago <- get_map(location = sbbox, zoom = 10, maptype = "terrain")
# Create map and plot each station colored according to the number of rides
chicagomap <- ggmap(chicago) +
  geom_jitter(data = mapping_data, aes(x = start_lng, y = start_lat, color = numtrips)) +
  scale_color_gradient(low = "blue", high = "red") +
  facet_wrap(~usertype) + 
  labs(title="Most popular stations", x = "longitude", y = "latitude") +
  theme(plot.title = element_text(size = 12, face = "bold")) +
  labs(color = "Number of rides") +
  theme(legend.title = element_text(size = 12, face = "bold"),
        legend.text = element_text(color = "black")) +
  theme(strip.text.x = element_text(face = "bold", size = 10))
chicagomap
```
<p align="center">
<img src="images/1_MostPopularStations.png" width="700" height="450">
</p>

The map shows that casual users and members roughly have the same
preferences for stations. Stations near the city center and shoreline
seem to attract the most attention, whereas only a small proportion of
users take advantage of stations in more residential areas in the north
or south of Chicago.

In the following, a dataframe is created that lists the top 10 most
frequently used stations by casual riders. This helps to verify trends
and single out certain stations that are suitable for intensified
marketing campaigns.

``` r
# Create a data frame that lists the top 10 most frequently used stations by casual riders
popular_stations_casual <- mapping_data %>%
  filter(usertype == "casual") %>%
  ungroup() %>% 
  arrange(desc(numtrips)) %>% 
  slice(1:10)
popular_stations_casual
```

    ## # A tibble: 10 × 5
    ##    start_station_name         start_lat start_lng usertype numtrips
    ##    <chr>                          <dbl>     <dbl> <chr>       <int>
    ##  1 Streeter Dr & Grand Ave         41.9     -87.6 casual      63704
    ##  2 Millennium Park                 41.9     -87.6 casual      31811
    ##  3 Michigan Ave & Oak St           41.9     -87.6 casual      28344
    ##  4 Shedd Aquarium                  41.9     -87.6 casual      22285
    ##  5 Theater on the Lake             41.9     -87.6 casual      20388
    ##  6 Lake Shore Dr & Monroe St       41.9     -87.6 casual      18880
    ##  7 Wells St & Concord Ln           41.9     -87.6 casual      18704
    ##  8 Clark St & Lincoln Ave          41.9     -87.6 casual      16135
    ##  9 Wells St & Elm St               41.9     -87.6 casual      15685
    ## 10 Indiana Ave & Roosevelt Rd      41.9     -87.6 casual      15638

``` r
# Visualize the location of these stations
sbbox <- make_bbox(lon = c(-87.7, -87.6), lat = c(41.95, 41.85), f = .05)
chicago_center <- get_map(location=sbbox, zoom=12, maptype="terrain")
chicago_centermap <- ggmap(chicago_center) +
  geom_point(data=popular_stations_casual, aes(x=start_lng,y=start_lat), color = "red", size = 2) +
  labs(title="Top 10 stations used by casual riders",
       x = "longitude",
       y = "latitude") +
  theme(plot.title = element_text(size = 12, face = "bold", hjust = 0.5))
chicago_centermap
```

<p align="center">
<img src="images/2_TopStationsUsedByCasuals.png" width="400" height="500">
</p>


The list of most popular stations for casual riders is topped by
**Streeter Dr & Grand Ave** station with over 60,000 trips in 2021. The
fact that all most popular stations are centered around the Navy Pier of
Chicago confirms the company’s initial analysis that most riders use the
service for leisure in a more touristy area.

<br>

### 4.3 Visualizing the number of trips

#### 4.3.1 By month

``` r
# Visualize the number of rides for each month
biketrips2021_v2 %>% 
  group_by(month, usertype) %>% 
  summarize(number_of_rides = n(), .groups = "keep") %>% 
  ggplot(aes(x = month,y=number_of_rides, color = usertype, group = usertype)) +
      geom_line() +
      geom_point() +
      theme_light() +
      labs(title="Number of rides during the year",
           x = "Month",
           y = "Number of rides") +
      theme(plot.title = element_text(size = 12), legend.title = element_blank()) +
      scale_y_continuous(labels = scales::comma)
```

<p align="center">
<img src="images/3_NumberOfRidesYear.png" width="700" height="500">
</p>


In general it can be observed that the summer months seem to be for both
– casual riders and members – the most popular time of the year to use
the bike service. Members seem to use the service a little more
constantly throughout the year, whereas casual riders predominantly use
the service during summer months. Hence unsurprisingly, the number of
casual rides only exceed the number of rides taken by members in exactly
these peak summer months.

It would be interesting to further analyze to what extent Covid-19
related *stay at home days* effected the number of rides, especially in
the winter months. On top of that, the data should be compared to
different years to verify if this pattern is consistent over the years.
<br>

#### 4.3.2 By weekday

``` r
# Visualize the number of rides for each weekday
biketrips2021_v2 %>% 
  group_by(usertype, weekday) %>% 
  summarize(number_of_rides = n(), .groups = "keep") %>% 
  ggplot(aes(x = weekday, y = number_of_rides, fill = usertype)) +
      geom_col(position = "dodge") +
      theme_light() +
      labs(title="Number of rides for each weekday",
       x = "Weekdays",
       y = "Number of rides") +
      theme(plot.title = element_text(size = 12), legend.title = element_blank())  +
      scale_y_continuous(labels = scales::comma)
```

<p align="center">
<img src="images/4_NumberOfRidesWeekday.png" width="700" height="500">
</p>

The bar chart shows that casual users prevailingly use the bike service
during weekends. This confirms the previously already stated assumption
that the vast majority of casual riders use the service for leisure
purposes. On the other hand, members seem to use the service in a more
balanced way throughout the week.

Interestingly, the most common weekday for members was *Wednesday*
whilst the most common day for casual riders was *Saturday*. This
further indicates that members are using the service for their every day
commute and other daily activities unlike casual riders whom seem to use
the service predominately for leisure activities. <br>

#### 4.3.3 By time of day

``` r
# Dataframe of all trips from midnight to noon
early_trips <- biketrips2021_v2 %>%
  filter(time_of_day >= "00:00:00" & time_of_day < "12:00:00")

# Dataframe of all trips from noon to midnight
late_trips <- biketrips2021_v2 %>%
  filter(time_of_day >= "12:00:00" & time_of_day < "24:00:00")

# Convert the "time of day" variable from string into date data type
early_trips$time_of_day <- as.POSIXct(early_trips$time_of_day, format = "%H:%M:%S")
late_trips$time_of_day <- as.POSIXct(late_trips$time_of_day, format = "%H:%M:%S")

# Group the time variable by hours
early_trips$by60 <- cut(early_trips$time_of_day, breaks = "60 mins")
late_trips$by60 <- cut(late_trips$time_of_day, breaks = "60 mins")

## Early
# Create a dataframe which counts the number of trips from midnight to noon for each usertype
circular_bar_chart_data_early <- early_trips %>%
  group_by(by60, usertype) %>%
  mutate(numtrips_000s = (n()/1000)) %>%
  distinct(by60, usertype, numtrips_000s)
# Create a circular bar chart to show the popularity of each hour
early <- ggplot(circular_bar_chart_data_early) +
  # Make a custom panel grid 
  geom_hline(aes(yintercept = y), data.frame(y = c(0:4) * 125), color = "lightgrey") + 
  # Create a stacked bar char
  geom_bar(aes(x = by60, y = numtrips_000s, fill = usertype), stat="identity") +
  # Create circular shape which starts in the mid-line  
  coord_polar(start = -0.25, direction = 1) + 
  ylim(-600, 500) +
  # Manually add x-axis labels 
  annotate(x = 1, y = -55, label = "00:00", geom = "text", size = 2) +
  annotate(x = 2, y = -65, label = "01:00", geom = "text", size = 2) +
  annotate(x = 3, y = -80, label = "02:00", geom = "text", size = 2) +
  annotate(x = 4, y = -95, label = "03:00", geom = "text", size = 2) +
  annotate(x = 5, y = -80, label = "04:00", geom = "text", size = 2) +
  annotate(x = 6, y = -65, label = "05:00", geom = "text", size = 2) +
  annotate(x = 7, y = -55, label = "06:00", geom = "text", size = 2) +
  annotate(x = 8, y = -65, label = "07:00", geom = "text", size = 2) +
  annotate(x = 9, y = -80, label = "08:00", geom = "text", size = 2) +
  annotate(x = 10, y = -95, label = "09:00", geom = "text", size = 2) +
  annotate(x = 11, y = -80, label = "10:00", geom = "text", size = 2) +
  annotate(x = 12, y = -65, label = "11:00", geom = "text", size = 2) +
  # Manually add y-axis scaling labels 
  annotate(x = 1, y = 160, label = "125.000", geom = "text", size = 2) +
  annotate(x = 1, y = 285, label = "250.000", geom = "text", size = 2) +
  annotate(x = 1, y = 410, label = "375.000", geom = "text", size = 2) +
  # Add title
  labs(title = "Number of rides from 00:00 - 12:00") +
  # Set light theme 
  theme_light() +
  # Adjust title and remove unnecessary labels 
  theme(plot.title = element_text(size = 10, hjust = 0.5), 
        axis.title = element_blank(), axis.ticks = element_blank(), 
        axis.text = element_blank(), legend.title = element_blank())


## Late
# Create data frame which counts the number of trips from noon to midnight for each usertype
circular_bar_chart_data_late <- late_trips %>%
  group_by(by60, usertype) %>%
  mutate(numtrips_000s = (n()/1000)) %>%
  distinct(by60, usertype, numtrips_000s)
# Create a circular bar chart to show the popularity of each hour
late <- ggplot(circular_bar_chart_data_late) +
  # Make custom panel grid 
  geom_hline(aes(yintercept = y), data.frame(y = c(0:4) * 125), color = "lightgrey") + 
  # Create a stacked bar char
  geom_bar(aes(x = by60, y = numtrips_000s, fill = usertype), stat = "identity") +
  # Create circular shape which starts in the mid-line  
  coord_polar(start = -0.25, direction = 1) + 
  ylim(-600, 500) +
  # Manually add x-axis labels 
  annotate(x = 1, y = -55, label = "12:00", geom = "text", size = 2) +
  annotate(x = 2, y = -65, label = "13:00", geom = "text", size = 2) +
  annotate(x = 3, y = -80, label = "14:00", geom = "text", size = 2) +
  annotate(x = 4, y = -95, label = "15:00", geom = "text", size = 2) +
  annotate(x = 5, y = -80, label = "16:00", geom = "text", size = 2) +
  annotate(x = 6, y = -65, label = "17:00", geom = "text", size = 2) +
  annotate(x = 7, y = -55, label = "18:00", geom = "text", size = 2) +
  annotate(x = 8, y = -65, label = "19:00", geom = "text", size = 2) +
  annotate(x = 9, y = -80, label = "20:00", geom = "text", size = 2) +
  annotate(x = 10, y = -95, label = "21:00", geom = "text", size = 2) +
  annotate(x = 11, y = -80, label = "22:00", geom = "text", size = 2) +
  annotate(x = 12, y = -65, label = "23:00", geom = "text", size = 2) +
  # Mannually add y-axis scaling labels 
  annotate(x = 12, y = 160, label = "125.000", geom = "text", size = 2, angle=30) +
  annotate(x = 12, y = 285, label = "250.000", geom = "text", size = 2, angle=30) +
  annotate(x = 12, y = 410, label = "375.000", geom = "text", size = 2, angle=30) +
  # Add title
  labs(title = "Number of rides from 12:00 - 24:00") +
  # Set light theme 
  theme_light() +
  # Adjust title and remove unnecessary labels
  theme(plot.title = element_text(size = 10, hjust = 0.5), 
        axis.title = element_blank(), axis.ticks = element_blank(), 
        axis.text = element_blank(), legend.title = element_blank())
```

<p align="center">
<img src="images/5_NumberOfRidesEarlyToLate.png" width="900" height="400">
</p>

The circular bar plots show that 5pm is the most popular time of day for
users to rent a bike. Particularly for members, a significant increase
of bike rides can be observed in the early morning around 7/8 am and the
early evening at around 5 pm. This further supports the hypothesis that
a lot of members use the bike sharing service for their work commutes.

On the other hand, casual riders don’t seem to use the service a lot
before 11 am. Thenceforth, they gradually catch up to the amount of
rides taken by members and even exceed them from 9 pm to 4 am. Combined
with the insights from the previous bar chart, showing that casual
riders especially use the bike service on weekends, this could suggest
that casual riders may be using the bikes as a ride home after a night
out instead of using the car or taking a taxi.

<br>

### 4.4 Trip length

In the following, I will investigate how casual riders and members use
the bike share system differently regarding ride length. I will divide
all bike rides into two dataframes containing data about short and long
trips. For this analysis step, I will define a short trip as a bike ride
with a duration that is equal or less than 30 minutes. Logically, every
trip longer than 30 minutes is defined as a long trip.

``` r
## All trips
# Calculate the ride distribution among users
all_trips_per <- biketrips2021_v2 %>% 
  group_by(usertype) %>%
  summarise(percent = n() / nrow(biketrips2021_v2))
all_trips_per$percent <- percent(all_trips_per$percent)

# Create a pie chart
all_trips_pie <- all_trips_per %>%
  ggplot(aes(x="", y=percent, fill = usertype)) +
    geom_bar(stat="identity", width=1, color="white") +
    coord_polar("y", start=0) +
    # Remove background, grid and numeric labels
    theme_void() +
    geom_label(aes(label = percent), position = position_stack(vjust = 0.5),
              show.legend = FALSE) +
    labs(title = expression(underline(All~trips))) +
    theme(plot.title = element_text(size = 14, hjust = 0.5), legend.title = element_blank())


### Short trips
# Create a dataframe containing all short trips
short_trips <- biketrips2021_v2 %>%
  filter(duration <= 1800)

# Calculate percentage distribution among usertypes
short_trips_per <- short_trips %>%
  group_by(usertype) %>%
  summarise(percent = n() / nrow(short_trips))
short_trips_per$percent <- percent(short_trips_per$percent)

# Calculate the percentage of short trips in relation to all trips
n <- nrow(short_trips)/nrow(biketrips2021_v2)
n_per <- percent(n)

# Create a pie chart
short_trips_pie <- short_trips_per %>%
  ggplot(aes(x="", y=percent, fill = usertype)) +
    geom_bar(stat="identity", width=1, color="white") +
    coord_polar("y", start=0) +
    # Remove background, grid and numeric labels
    theme_void() +
    geom_label(aes(label = percent), position = position_stack(vjust = 0.5),
               show.legend = FALSE) +
    labs(title = expression(underline(Short~trips~(under~30~minutes))), caption=paste0(n_per, 
    " of all trips are short")) +
    theme(plot.title = element_text(size = 14, hjust = 0.5), legend.position="none")


### Long trips
# Create a dataframe containing all long trips
long_trips <- biketrips2021_v2 %>%
  filter(duration > 1800)

# Calculate percentage distribution among usertypes
long_trips_per <- long_trips %>%
  group_by(usertype) %>%
  summarise(percent = n() / nrow(long_trips))
long_trips_per$percent <- percent(long_trips_per$percent)

# Calculate the percentage of long trips in relation to all trips
z <- nrow(long_trips)/nrow(biketrips2021_v2)
z_per <- percent(z)

# Create pie chart
long_trips_pie <- long_trips_per %>%
  ggplot(aes(x="", y=percent, fill = usertype)) +
    geom_bar(stat="identity", width=1, color="white") +
    coord_polar("y", start=0) +
    # Remove background, grid and numeric labels
    theme_void() +
    geom_label(aes(label = percent), position = position_stack(vjust = 0.5),
               show.legend = FALSE) +
    labs(title = expression(underline(Long~trips~(over~30~minutes))), caption=paste0(z_per, 
    " of all trips are long")) +
    theme(plot.title = element_text(size = 14, hjust = 0.5), legend.position="none")
```

<p align="center">
<img src="images/6_TripLengthAll.png" width="500" height="400">
</p>

<p align="center">
<img src="images/7_TripLengthShortLong.png" width="700" height="400">
</p>


The first pie chart shows that 55.23% of all trips were taken by members
and only 44.77% by casual riders. However, this proportion changes when
trips are categorized into short and long. On the one hand, short trips
under 30 minutes are even more predominantely used by members (60.69%),
whereas on the other hand longer trips are clearly dominated by casual
riders (74.12%).

These insights aline with the results gained in section 4.1 from the
descriptive analysis, which showed that the average ride length of
casual users (29.03 minutes) is over twice as long as the average ride
length of members (13.38 minutes). Again, this further strengthens the
idea that casual users ride the bikes for leisure purposes.

<br>

### 4.5 Bike type preference

``` r
ggplot(biketrips2021_v2, aes(x = rideable_type, fill=usertype)) +
  geom_bar()+
  theme_light()+
  facet_wrap(~usertype) +
  labs(title="Bike type preferences", x = "Bike type", y = "Number of rides") +
  theme(plot.title = element_text(size = 12, face = "bold"),legend.position = "None")
```

<p align="center">
<img src="images/8_BikeType.png" width="700" height="500">
</p>


The bar chart shows that both usertypes roughly have the same
preferences for bike types. Both annual members and casual riders
favourite option is the ‘classic’ bike, followed by the ‘electric’ bike
and the least favourite ‘docked’ option.

Interestingly, for 2021 there was only one ride recorded of a member
using the docked bike. Further efforts are needed to clarify the
occurrence of such an anomaly. More information regarding the rider
would also be beneficial for a more substantive analysis.

<br>

## 5. Recommendations

Before we dive into the conclusion and focus on the main recommendations
and takeaways of this case study, lets have a look at the different
pricing options ‘Cyclistic’ ([Divvy
Bikes](https://divvybikes.com/pricing)) offers as of January 2022:

-   Single ride, $3.30/trip
-   Day pass, $15/day
-   Annual memberships, $9/month

Users can choose between a singe ride ticket of one trip up to 30
minutes, a day pass with unlimited 3 hour rides for 24 hours and an
annual membership option for unlimited 45 minute rides.

As identified in the problem statement, the final marketing
recommendations should not be focused on encouraging new customers to
use Cyclistic’s bike service but instead focus on incentivizing casual
riders to convert into annual members.

After concluding the analysis I came up with the following three
recommendations:

**1. Notify users about the price benefit of annual memberships** <br> 
A closer look at the pricing model reveals that a user already saves money if he/she uses the service on average 3 times a month. From the registration process the company should be in possession of each users address information, email and even phone number. They should use this asset to notify customers via emails, app notifications or other (digital) ways and highlight the longterm pricing advantages from investing into an annual membership rather than buying single ride tickets.

**2. Consider alternative membership options and pricing models** <br>
Another very interesting takeaway from this case study is how differently the service is being used during the year. Casual riders clearly peak in the summer months but avoid the service during colder seasons. As we've seen above, Cyclistic only provides an annual membership option. Since some casual riders probably know that they won't use the service during winter months, a more flexible membership option with a time span of for example 3 to 6 months could help to tie these customers to the company in the long run. In addition, one could think about different pricing models for different seasons. Overall we've discovered quite a big gap in demand comparing winter and summer usage. Therefore, I propose to discount memberships in colder months to make them more attractive.

**3. Highlight benefits of cycling in times of Covid-19** <br> Last but
not least, Cyclistic could also focus their marketing campaign on
potential health benefits of exercising regularly outdoors in the fresh
air. Especially in times of Covid-19 they should highlight the
advantages of outdoor cycling compared to usually poorly ventilated
means of transport like cars or public buses. For their efforts they
should target stations which are frequently used by casual riders. In
section 4.2 I revealed which stations are most suitable for this. These
included *Streeter Dr & Grand Ave*, *Millennium Park* and *Michigan Ave
& Oak St*, just to name the three most popular ones.

<br>

## Next steps

While cleaning and analyzing the data I was confronted with a few
anomalies. As an example I could name the large number of blank
start/end station names, inconsistent station IDs, negative ride lengths
or the fact that within 12 months there was only one bike ride recorded
of a member using a ‘docked’ bike. In order to ensure the data’s
meaningfullness it is important to clarify these anomalies in future
steps.

Apart from that, it is essential to gather and include more data into
the analysis. So far only bike trip data from 2021 has been analyzed. To
really reveal patterns and gain more substantive insights it is crucial
to do a multiple year comparison. In this regard, it would be
interesting to see to what extent Covid-19 affected the bike usage.
