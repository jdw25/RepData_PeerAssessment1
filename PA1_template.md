# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
*Load the data (i.e. read.csv()). Process/transform the data (if necessary) into a format suitable for your analysis*

```r
# Load packages to be used
library(dplyr)
library(reshape2)
library(lubridate)
library(ggplot2)
```



```r
# Read in the data
data <- read.csv(unzip("activity.zip"),stringsAsFactors=FALSE)
```

## What is mean total number of steps taken per day?
Note: Ignoring the missing values in the dataset.

*(1) Calculate the total number of steps taken per day*

```r
daily_summary <- 
    summarize(group_by(data,date),
              sum_steps=sum(steps,na.rm=T)
              )
```

*(2) Make a histogram of the total number of steps taken each day*

```r
qplot(daily_summary$sum_steps, binwidth=500)
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

*(3) Calculate and report the mean and median of the total number of steps taken per day*

```r
mean(daily_summary$sum_steps, na.rm=T)
```

```
## [1] 9354.23
```

```r
median(daily_summary$sum_steps, na.rm=T)
```

```
## [1] 10395
```

## What is the average daily activity pattern?
*(1) Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)*

```r
interval_summary <- 
    summarize(group_by(data,interval),
              mean_steps=mean(steps,na.rm=T),
              sum_steps=sum(steps,na.rm=T))

qplot(interval,mean_steps,data=interval_summary, geom = "line")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

*(2) Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?*

```r
interval_summary$interval[which.max(interval_summary$mean_steps)]
```

```
## [1] 835
```


## Imputing missing values
*(1) Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)*

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

*(2) Devise a strategy for filling in all of the missing values in the dataset.*

Here I decided use the mean for that 5-minute interval across all days in the dataset (excluding missing values).

*(3) Create a new dataset that is equal to the original dataset but with the missing data filled in.*

Simply add a column to the dataframe to hold the original and imputed values where was missing

```r
data <-
    data %>% 
    group_by(interval) %>%
    mutate(steps_impute= replace(steps, is.na(steps), mean(steps, na.rm=TRUE)))
```

*(4) Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.*

```r
daily_summary <- 
    summarize(group_by(data, date),
              sum_steps=sum(steps,na.rm=T),
              sum_steps_impute=sum(steps_impute,na.rm=T)
              )

#reshape in "long" format ggplot can work with
daily_summary_melt <- melt(daily_summary, id="date")

# generate a paneled histogram, so can view side-by-side
qplot(value, data=daily_summary_melt, facets = . ~ variable, binwidth=500)
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

```r
#
mean(daily_summary$sum_steps)
```

```
## [1] 9354.23
```

```r
mean(daily_summary$sum_steps_impute)
```

```
## [1] 10766.19
```

```r
median(daily_summary$sum_steps)
```

```
## [1] 10395
```

```r
median(daily_summary$sum_steps_impute)
```

```
## [1] 10766.19
```

*Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?*

Yes, we can see visually from the histograms and comparison of mean and median values before and after imputation that replacing missing data with imputed values

- reduces the number of days with 0 steps
- shifts those days to the 11000-11500 range
- increases the daily total and average number of steps (of course!).  
- the mean and median daily steps become equivalent, meaning a (more) symmetrical distribution. 

## Are there differences in activity patterns between weekdays and weekends?

```r
# Leverage lubridate to get weekday, then create a new variable to store "weekday" vs "weekend"
data$date <- ymd(data$date)
data$weekday <- wday(data$date, label=T)
data$daytype = ifelse(data$weekday %in% c('Sun','Sat'), 'Weekend','Weekday')
```

*Make a paneled time-series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)*

```r
daytype_summary <-
    summarize(
        group_by(data,daytype,interval),
        sum_steps=sum(steps,na.rm=T),
        sum_steps_impute=sum(steps_impute,na.rm=T),
        mean_steps=mean(steps,na.rm=T),
        mean_steps_impute=mean(steps_impute,na.rm=T)
    )

#reshape in "long" format ggplot can work with
daytype_summary_melt <- melt(daytype_summary, id=c("daytype","interval"))

# generate a paneled histogram, so can view side-by-side
qplot(interval, value, data=daytype_summary_melt, geom = "line", facets = . ~ daytype)
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

Basically, we see that the weekend involves fewer steps, and there is not the morning "spike" 8am-10am.
