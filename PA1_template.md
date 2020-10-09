---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data


```r
data <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?

1. Calculate the total number of steps taken per day

```r
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
data <- group_by(data, date)
totals <- summarise(data, Total.Steps=sum(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
head(totals)
```

```
## # A tibble: 6 x 2
##   date       Total.Steps
##   <chr>            <int>
## 1 2012-10-01          NA
## 2 2012-10-02         126
## 3 2012-10-03       11352
## 4 2012-10-04       12116
## 5 2012-10-05       13294
## 6 2012-10-06       15420
```

2. Make a histogram of the total number of steps taken each day

```r
hist(totals$Total.Steps, xlab="Total Steps", main="Histogram of Total Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

3. Calculate and report the mean and median of the total number of steps taken per day

```r
mean(totals$Total.Steps, na.rm=TRUE)
```

```
## [1] 10766.19
```

```r
median(totals$Total.Steps, na.rm=TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
data <- group_by(data, interval)
averages <- summarise(data, Average.Steps=mean(steps, na.rm=TRUE))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
with(averages, plot(interval, Average.Steps, type="l"))
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
averages[which.max(averages$Average.Steps),]
```

```
## # A tibble: 1 x 2
##   interval Average.Steps
##      <int>         <dbl>
## 1      835          206.
```

## Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
## filling in missing values using the mean for that 5-minute interval
impdata <- data
for (i in 1:nrow(impdata)) {
  if (is.na(impdata$steps[i])) {
    impdata$steps[i] <- averages$Average.Steps[which(averages$interval==impdata$interval[i])]
  }
}
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
impdata <- group_by(impdata, date)
imptotals <- summarise(impdata, Total.Steps=sum(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
hist(imptotals$Total.Steps, xlab="Total Steps", main="Histogram of Total Steps (with Imputed Missing Values)")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

```r
mean(imptotals$Total.Steps)
```

```
## [1] 10766.19
```

```r
median(imptotals$Total.Steps)
```

```
## [1] 10766.19
```

## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
impdata$Day.Type <- weekdays(as.Date(impdata$date))
impdata$Day.Type <- sub("Monday|Tuesday|Wednesday|Thursday|Friday", "Weekday", impdata$Day.Type)
impdata$Day.Type <- sub("Saturday|Sunday", "Weekend", impdata$Day.Type)
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
library(ggplot2)
impdata <- group_by(impdata, Day.Type, interval)
impaverages <- summarise(impdata, Average.Steps=mean(steps))
```

```
## `summarise()` regrouping output by 'Day.Type' (override with `.groups` argument)
```

```r
par(mfrow=c(2,1))
qplot(interval, Average.Steps, data=impaverages, facets=Day.Type~., geom="line")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->
