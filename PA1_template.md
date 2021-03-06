# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
Directly download the dataset and load it accordingly. 


```r
url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
temp <- tempfile()
download.file(url, temp, method = "curl")
data <- read.csv(unz(temp, "activity.csv"))
unlink(temp)
```

## What is mean total number of steps taken per day?
### 1. Histogram of the total number of steps taken each day


```r
# spd: steps per day
spd <- aggregate(steps ~ date, data, sum)$steps
hist(spd,
     main = "The total number of steps taken per day",
     xlab = "Steps", ylab = "Frequency (day)",
     breaks = 10)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

### 2. The mean and median of the total number of steps taken per day


```r
mean.spd <- mean(spd)
median.spd <- median(spd)
```

The mean of the total number of steps is 10766 and the median is 10765.


## What is the average daily activity pattern?
### 1. Time series plot of the 5-minute interval and the average number of steps taken.

```r
# sbi: steps by interval
sbi <- aggregate(steps ~ interval, data, mean)
plot(sbi$steps, type = "l",
     main = "Average daily activity pattern",
     xlab = "5-min interval", ylab = "Averaged number of steps")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
max.interval = which.max(sbi$steps)
max.interval
data$interval[max.interval]
```

The maximum number of steps is 104 which is contained in the interval 835.



## Imputing missing values

### 1. The total number of missing values in the dataset


```r
sum(is.na(data))
```

The total number of missing values in the dataset is 2304.


### 2. Filling in all of the missing values in the dataset. 

The missing values were replaced by the mean value for its 5-minute interval.

### 3. A new dataset with the missing data filled in.


```r
imputation <- function(steps, interval) {
        temp.data <- NA
        if (!is.na(steps)) temp.data <- c(steps)
        else temp.data <- (sbi[sbi$interval == interval, "steps"])
        return (temp.data)
}

new.data <- data
new.data$steps <- mapply(imputation, new.data$steps, new.data$interval)
```

### 4. Analyze the new data

#### a. Histogram of the total number of steps taken each day 


```r
new.spd <- aggregate(steps ~ date, new.data, sum)$steps
hist(new.spd,
     main = "The total number of steps taken per day",
     xlab = "Steps", ylab = "Frequency (day)",
     breaks = 10)
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 

#### b. The mean and median total number of steps taken per day. Compare to the data with missing values. What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
new.steps.mean <- mean(new.spd)
new.steps.median <- median(new.spd)
```

The mean of the total number of steps is 10766 and the median is 10766. The mean is same, but median is different. It is because missing values were filled in the average of its interval.


## Differences in activity patterns between weekdays and weekends

### 1. Dataset with new factor variable "weekdays" -- indicating weekday or weekend


```r
library(chron)
new.data$week<-factor(is.weekend(new.data$date), 
                      levels=c(T, F), labels=c("Weekend", "Weekday"))
```

### 2. Plot of the 5-minute interval and the average number of steps taken.


```r
new.sbi <- aggregate(steps ~ interval + week, new.data, mean)

library(ggplot2)
ggplot(new.sbi, aes(interval, steps)) + geom_line() + facet_grid(week ~ .) + xlab("5-minute interval") + ylab("Averaged number of steps")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 

The subject looks to walk more often on weekends than weekdays. 
