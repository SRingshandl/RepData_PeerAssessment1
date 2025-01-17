---
title: "Reproducible Research: Peer Assessment 1"
author: "SRingshandl"
date: "2022-09-09"
output:
  html_document:
    keep_md: true
---



## Loading and preprocessing the data

First we have to load our data.



```r
data <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?

We're ignoring missing values for this step so removal of those comes first.  
Afterwards we sum up all steps on each day and create a histogram out of our filtered data.


```r
data_no_NAs <- data[!is.na(data$steps),]
sum_per_day <- tapply(data_no_NAs$steps, data_no_NAs$date, sum)
hist(sum_per_day, main = "Histogram of steps per day", xlab = "Steps per day")
```

![](PA1_template_files/figure-html/Histogram_of_steps_per_day-1.png)<!-- -->

Next is the report on the mean value and the median of steps done per day.


```r
mean(sum_per_day)
```

```
## [1] 10766.19
```

```r
quantile(sum_per_day,0.5)
```

```
##   50% 
## 10765
```

## What is the average daily activity pattern?

We're calculating the average steps in each time interval and display the data in a line plot.

```r
interval_mean_steps <- tapply(data_no_NAs$steps, data_no_NAs$interval, mean)
plot(names(interval_mean_steps), interval_mean_steps, 
     type = "l",
     main = "Mean steps per interval",
     xlab = "Interval start minute",
     ylab = "mean steps during interval")
```

![](PA1_template_files/figure-html/Mean_steps_per_interval-1.png)<!-- -->

To answer the question which 5 minute interval contains the most steps averaged across all days we do this:


```r
interval_mean_steps[which(interval_mean_steps == max(interval_mean_steps))]
```

```
##      835 
## 206.1698
```

## Imputing missing values

With the following command we can count how many rows are complete.


```r
table(complete.cases(data))
```

```
## 
## FALSE  TRUE 
##  2304 15264
```

We see that 15264 rows are complete (which is the same amount as the dataset where we removed rows with NAs --> only misssing values in the column called *steps*). Also we have 2304 cases with missing values.
For imputation of these values we will use the previously calculated mean of that interval taken from other days. We store the imputed data in a variable called *data_imputed*.


```r
data_imputed <- data
for(i in 1:nrow(data)){
  if(is.na(data$steps[i])){
    interval <- data$interval[i]
    data_imputed$steps[i] <- interval_mean_steps[which(names(interval_mean_steps) == interval)]
  }
}
```

When we create a histogram, the mean and median as before, this time using the imputed data, we see the same results. We used the average values of intervals with data for NAs, and as for the calculation of those NAs were not considered we now have more (of the same) data.


```r
sum_per_day_imputed <- tapply(data_imputed$steps, data_imputed$date, sum)
hist(sum_per_day_imputed, main = "Histogram of steps per day (imputed)", xlab = "Steps per day")
```

![](PA1_template_files/figure-html/Histogram_of_steps_per_day_imputed-1.png)<!-- -->

```r
mean(sum_per_day)
```

```
## [1] 10766.19
```

```r
quantile(sum_per_day,0.5)
```

```
##   50% 
## 10765
```



## Are there differences in activity patterns between weekdays and weekends?

To split the plot that we created in the chaper "What is the average daily activity pattern?" we have to add another row to our imputed dataset that specifies whether data originates from a weekday or a weekend. We use the weekdays function to translate our dates into wekkday names like *monday*. After that we compare these days to two lists. One for weekdays and another one for the days that fall on a weekend. Then we create two subsets. One for the weekdays and another one for weekend data, Lastly we calculate the mean steps in each interval for each of those subsets.


```r
#I'm not in the US. Days would be confusing so this sets locale to the US
Sys.setlocale("LC_TIME", "C")
data_imputed$weekday <- weekdays(as.Date(data_imputed$date))
days_weekday <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
days_weekend <- c("Saturday", "Sunday")
data_imputed$day_group <- NA

for(i in 1:nrow(data_imputed)){
  if(data_imputed$weekday[i] %in% days_weekday)  data_imputed$day_group[i] <- "weekday"
  if(data_imputed$weekday[i] %in% days_weekend)  data_imputed$day_group[i] <- "weekend"
}

subset_weekdays <- subset(data_imputed, data_imputed$day_group == "weekday")
subset_weekend <- subset(data_imputed, data_imputed$day_group == "weekend")

interval_mean_steps_imputed_weekday <- tapply(subset_weekdays$steps, subset_weekdays$interval, mean)
interval_mean_steps_imputed_weekend <- tapply(subset_weekend$steps, subset_weekend$interval, mean)
```

Lastly we have to plot the data. Lattice nicely splits data according to groups so we'll use that. We need to specify a usable datadrame for that so  we create ist out of the previouly prepared subsets of weekday and weekend data. As the column Interval isn't numeric we change that, And finally we plot the data.


```r
library(lattice)

df_interval_days <- data.frame(INTERVAL = rep(names(interval_mean_steps_imputed_weekday),2),
                               MEAN_STEPS = c(interval_mean_steps_imputed_weekday, 
                                              interval_mean_steps_imputed_weekend),
                               GROUP = c(rep("weekday", length(interval_mean_steps_imputed_weekday)),
                                         rep("weekend", length(interval_mean_steps_imputed_weekend))))

df_interval_days$INTERVAL <- as.numeric(df_interval_days$INTERVAL)

xyplot(MEAN_STEPS ~ INTERVAL | GROUP, data = df_interval_days,
       type = "l", 
       ylab = "step count",
       xlab = "Interval start minute",
       main = "Step count split by day group")
```

![](PA1_template_files/figure-html/Step_count_split_by_day_group-1.png)<!-- -->

Analysis done!
