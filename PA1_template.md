---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---
The loading of all the packages we'll need is just below:

```r
library(stringr)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(lubridate)
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following objects are masked from 'package:base':
## 
##     date, intersect, setdiff, union
```

```r
library(ggplot2)
```


## Loading and preprocessing the data

The data file is available in the course's assignment.
Make sure you download it and place it in your working directory before you start.

To know where your working directory is:

```r
getwd()
```

```
## [1] "C:/Users/Me/Documents/R/RepData_PeerAssessment1"
```

The activity.CSV file should be in this directory. You can now load it into a dataframe data:

```r
data <- read.csv("activity.csv")
```

Let's see what data looks like:

```r
summary(data)
```

```
##      steps            date              interval     
##  Min.   :  0.00   Length:17568       Min.   :   0.0  
##  1st Qu.:  0.00   Class :character   1st Qu.: 588.8  
##  Median :  0.00   Mode  :character   Median :1177.5  
##  Mean   : 37.38                      Mean   :1177.5  
##  3rd Qu.: 12.00                      3rd Qu.:1766.2  
##  Max.   :806.00                      Max.   :2355.0  
##  NA's   :2304
```

```r
head(data)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

```r
tail(data)
```

```
##       steps       date interval
## 17563    NA 2012-11-30     2330
## 17564    NA 2012-11-30     2335
## 17565    NA 2012-11-30     2340
## 17566    NA 2012-11-30     2345
## 17567    NA 2012-11-30     2350
## 17568    NA 2012-11-30     2355
```

We can see the format of the interval is specific: actually, it's a time format (as 2355 for 11:55PM) but the first time of the day is incomplete (0 or 5 for 00:00 or 00:05). We need to coerce the interval as a character with zeros in front of the value to have a valid time format:

```r
data$interval <- str_pad(as(data$interval, "character"), 4, side = "left", pad = "0")
data <- mutate(data, day_time = fast_strptime(data$interval, "%H%M"))

head(data)
```

```
##   steps       date interval            day_time
## 1    NA 2012-10-01     0000 0000-01-01 00:00:00
## 2    NA 2012-10-01     0005 0000-01-01 00:05:00
## 3    NA 2012-10-01     0010 0000-01-01 00:10:00
## 4    NA 2012-10-01     0015 0000-01-01 00:15:00
## 5    NA 2012-10-01     0020 0000-01-01 00:20:00
## 6    NA 2012-10-01     0025 0000-01-01 00:25:00
```

=> looks better!

## What is mean total number of steps taken per day?

We create a new dataframe data_per_day, grouping the data by day of measurement:

```r
data_per_day <- data %>%
     select(-interval) %>%
     group_by(date) %>%
     summarize(steps = sum(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
hist(data_per_day$steps, breaks = 15, xlab = "Total number of steps by day",
     main = "Histogram of total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
nb_steps_mean <- mean(data_per_day$steps, na.rm = TRUE)
nb_steps_median <- median(data_per_day$steps, na.rm = TRUE)
```
The mean of the total number of steps taken per day is 1.0766189\times 10^{4} and the median is 10765.

## What is the average daily activity pattern?

This time, we need to take an average of the number of steps for each 5 minutes interval. So we group the data differently, in a dataframe data_per_time:

```r
data_per_time <- data %>%
     select(-date) %>%
     group_by(interval) %>%
     summarize(day_time = min(day_time), steps = mean(steps, na.rm = TRUE))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
#qplot(data_per_time$day_time, data_per_time$steps, geom = 'path', xlab = "Time", ylab = "Number of steps",
#      main = "Time series plot of the average number of steps taken", scale_x_datetime(date_labels = "%H:%M"))

g <- ggplot(data_per_time, aes(day_time, steps))
g + geom_path() + scale_x_datetime(date_labels = "%H:%M") + labs(x = "Time", y = "Number of steps") + labs(title = "Average number of steps taken")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->


## Imputing missing values



## Are there differences in activity patterns between weekdays and weekends?
