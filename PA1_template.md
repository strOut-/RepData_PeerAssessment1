---
title: "Report on personal activity patterns"
author: "Chris Daly"
date: "Monday, August 11, 2014"
output:
  html_document:
    theme: spacelab
---

## Introduction
It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This report makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

### Libraries
The following libraries were used throughout the code.
```{r}
library(knitr)
library(lattice)
library(xtable)

```
```{r setoptions, echo = FALSE}
opts_chunk$set(eval = TRUE)
```

### Loading and preprocessing the data
A zip file contatining the data was downloaded from Amazon's cloudfront on the 11/08/2014 into a data folder in the working directory. 

```{r, eval = FALSE}
# check if a data folder exists; if not then create one
if (!file.exists("data")) {dir.create("data")}

# file URL and destination file
fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
destfile <- "./data/activity.zip"

# download the file and note the time
download.file(fileUrl, destfile = destfile)
dateDownloaded <- date()

```

The relevant csv file was then unzipped and loaded to R

```{r}
# from the zip file, read out the containing csv file
data_ <- read.csv(unz("./data/activity.zip", "activity.csv"))
```

### 1 - What is mean total number of steps taken per day?

The three columns in the data was then broken up into three vectors: date, steps, interval. A filter was created to isolate non-NA values from the data.

```{r}
# assign variables to the columns
steps <- data_$steps
date <- data_$date
interval <- data_$interval

# filter to isolate non-NA values
filter <- !is.na(steps)

# apply the steps filter to the date vector
filter_steps <- steps[filter]
filter_date <- date[filter]
```

A factor vector was created to distinguish each date. This vector was used to calculate the total number of steps for each day using the tapply function. A histogram of the total number of steps per day was plotted with 11 bins.

``` {r}
# create a factor vector for the non-NA days
days_factor <- factor(filter_date)

# get the total number of steps for each day
total_steps <- tapply(filter_steps, days_factor, FUN = sum)

# plot a histogram of the total number of steps taken each day
histogram(total_steps, breaks = 10, 
          xlab = "Total number of steps per day", 
          main = "Distribution of total steps per day", 
          col = "lightblue", 
          type = "count")
```

The mean and median were also calculated.

```{r}
mean_original <- mean(total_steps)
```
```{r, echo = FALSE}
mean_original
```
```{r}
median_original<- median(total_steps)
```
```{r, echo = FALSE}
median_original
```

### 2 - What is the average daily activity pattern?

A factor variable for the time intervals was created. The average steps were calculated using the tapply function on the factor variable and then rounded to two decimal places. A scale for the x axis was created to clearly show the values. A time series was plotted of the average steps vs the time interval.

```{r}
# create a factor vector for the time intervals
interval_factor <- factor(interval)
levels <- nlevels(interval_factor)
interval_factor <- factor(interval)[1:levels]

# calculate the average number of steps for each 5 minute period
average_steps <- tapply(steps, factor(interval), FUN = mean, na.rm = TRUE)
average_steps <- sapply(average_steps, simplify = array, round, 2)

scales=list( x=list(at = seq(0, 2400, 200)))     
   
# plot the time series
xyplot(as.numeric(average_steps) ~ interval[1:288], 
       type = "l", 
       xlab = "Time interval",
       ylab = "Average steps", 
       main = "Time series - average steps vs time interval", 
       scales = scales)

```

Then a data frame was constructed of the average steps and time intervals. The data frame was then sorted by average steps to get the maximum and the associated time interval was taken.

```{r}
# create a data frame of average steps and time interval
df_steps_interval <- data.frame(interval_factor, average_steps)

# sort df to get the row with the maximum amount of average steps
df_steps_interval <- df_steps_interval[order(df_steps_interval$average_steps, 
                                             decreasing = TRUE),]

# the first row contains the relevant time interval
time_interval_max <- df_steps_interval$interval_factor[1]

# convert the factor to a character and then to numeric
time_interval_max <- as.numeric(as.character(time_interval_max))
```
```{r, echo = FALSE}
time_interval_max
```

### 3 - Imputing missing values

There were a lot of NA values in the data set for the number of steps. The number was computed as follows:

```{r}
# number of NA values in original dataset
length(steps[is.na(steps)])
```

These null entries were given new values based on the corresponding 5 minute interval for the average steps that was computed previously. This was achieved by copying the original steps data into a new vector. The NA values for this vector were found using sapply and then they were looped over, replacing each one by the corresponding value in the average steps data.

```{r}
# take a copy of the original steps vector
new_steps <- steps

# fill in each NA value by taking the average for that time interval
for (i in which(sapply(new_steps, is.na))) {
  
  # set the value to the equivalent value in the average vector
  if (i <= 288){
    new_steps[i] <- average_steps[i]
  } 
  
  # wrap around 288 (avg time only has 24 hours of data) and add one because 
  # R is non-zero index
  else{
    j <- i%%288 + 1
    new_steps[i] <- average_steps[j]
  }
}
```

The new vector was factored by day and its steps were summed. A histogram of the total number of steps per day with imputted values was plotted with 11 bins.

```{r}
# create a factor vector for all of the days
new_days_factor <- factor(new_steps)

# get the total number of steps for each day
new_total_steps <- tapply(new_steps, new_days_factor, FUN = sum)

# plot a histogram of the total number of steps taken each day
histogram(new_total_steps, breaks = 10, 
          xlab = "Total number of steps per day", 
          main = "Distribution of total steps per day after imputted values", 
          col = "lightblue",
          type = "count")
```

The mean and median were also calculated.


```{r}
mean_new <- mean(new_total_steps)
```
```{r, echo = FALSE}
mean_new
```
```{r}
median_new <- median(new_total_steps)
```
```{r, echo = FALSE}
median_new
```
```{r, results='asis', echo = FALSE}
original <- c(mean_original, median_original)
new_ <- c(mean_new, median_new)
table <- data.frame(original, new_)
result <- apply(table, 1, function(x) (x[2])/(x[1]/100))
table$compare <- result
rownames(table)<-c("mean", "median")
print(xtable(table), type="html")

```

These values are drastically different from the original dataset; the new values are only about 6-7% that of the original ones. The impact of imputting average values for missing values ultimately had the effect of adding many more small values such as zeros and introduced a lot of bias into the dataset.

### 4 - Are there differences in activity patterns between weekdays and weekends?

The date vector was converted from a factor type to a date. It was then run through the weekdays() function to determine which day each date fell into. Using this data a factor was constructed for weekdays and weekends. A time series was plotted of the number of steps vs time interval for both weekdays and weekends.

```{r}
# convert the date vector from factor type to date
date_new <- as.Date(date)

# determine the day of the week for each date
whichDay <- weekdays(date_new)

# weekend day vector to compare with
weekendDays <- c("Saturday", "Sunday")

# construct a DF for these 4 values
DF <- data.frame(date_new, interval_factor, new_steps, whichDay)

# add a logical column to indicate whether a day ot type weekend/weekday
isWeekend <- DF$whichDay %in% weekendDays

# convert isWeekend to a factor variable
DF$dayType = factor(isWeekend,labels = c("Weekday","Weekend"))

# plot the time series
xyplot(DF$new_steps ~ interval | DF$dayType, layout = c(2, 1), type = "l", 
       xlab = "Time interval", ylab = "Number of steps", main = "Time series of number of steps vs time interval" )
```
