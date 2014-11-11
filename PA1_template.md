# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

1. Load the data (i.e. read.csv())
2. Process/transform the data (if necessary) into a format suitable for your analysis

```r
unzip("activity.zip")
part1_df <- read.csv("activity.csv", colClasses = c("numeric", "Date", "numeric"))
```


## What is mean total number of steps taken per day?

1. Make a histogram of the total number of steps taken each day

```r
library(dplyr, warn.conflicts = F)

part2_df <- part1_df %>%
    filter(complete.cases(steps)) %>%
    group_by(date) %>%
    summarise(total = sum(steps))

with(part2_df,
     hist(total,
          xlab = "Steps",
          main = "Total Number of Steps Taken Each Day"))
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png) 

2. Calculate and report the mean and median total number of steps taken per day

```r
mean(part2_df$total)
```

```
## [1] 10766.19
```

```r
median(part2_df$total)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
part3_df <- part1_df %>%
    group_by(interval) %>%
    summarise(avg = mean(steps, na.rm = T))

with(part3_df,
     plot(interval, avg,
          type = "l",
          ylab = "Steps",
          main = "Average Number of Steps Taken, Averaged Across All Days"))
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
as.numeric(part3_df[which.max(part3_df$avg), "interval"])
```

```
## [1] 835
```


## Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
part4_missing <- is.na(part1_df$steps)
sum(part4_missing)
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

The strategy we devise is to replace all NA values for each interval with the average value for that interval averaged accross all days, as previously calculated.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
part4_df <- part1_df
part4_df[part4_missing, "steps"] <- part3_df[, "avg"]
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
part4_total <- part4_df %>%
    group_by(date) %>%
    summarise(total = sum(steps))

with(part4_total, hist(total,
                       xlab = "Steps",
                       main = "Total Number of Steps Taken Each Day, Imputed"))
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png) 


```r
mean(part4_total$total)
```

```
## [1] 10766.19
```

```r
median(part4_total$total)
```

```
## [1] 10766.19
```

The mean is the same as the estimate from the first part of the assignment.  We are not surprised by this because our strategy for filling in missing steps was to use the average number of steps for each interval. The median is now approaching the mean. The impact of imputing missing data on the estimates of the total daily number of steps is minimal in this case.

## Are there differences in activity patterns between weekdays and weekends?


1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
part5_df <- part4_df %>%
    mutate(day = weekdays(date), 
           day = as.factor(ifelse(day == "Saturday" | day == "Sunday", "weekend", "weekday")))

part5_avg <- part5_df %>%
    group_by(interval, day) %>%
    summarise(avg = mean(steps))
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
library(lattice)
xyplot(avg ~ interval | day, part5_avg,
       type = "l",
       layout = c(1, 2),
       xlab = "Interval",
       ylab = "Number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png) 
