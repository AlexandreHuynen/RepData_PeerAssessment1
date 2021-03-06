# Reproducible Research: Peer Assessment 1
Alexandre Huynen  
3/12/2017  




## Loading and preprocessing the data

We start by unzipping and loading the data:


```r
if(!file.exists("activity.csv")){
        tmp <- tempfile()
        download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", tmp, method = "curl")
        unzip(tmp)
        unlink(tmp)
}

activity <- read.csv("activity.csv")

str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

The variable ```date``` being encodded as a ```Factor```, we convert it to the proper ```Date``` format:


```r
activity$date <- ymd(activity$date)
```

## What is the mean total number of steps taken per day?

To make a histogram of the total number of steps taken each day, it is necessary to first process the data and group it by day. In the following ```R``` code, this is done using the ```dplyr``` package. The mean and median total number of steps taken per day are also calculated before plotting these results.


```r
daily_activity <- activity %>% group_by(date) %>% 
        summarise(steps = sum(steps, na.rm = TRUE))

mean_steps <- mean(daily_activity$steps)
median_steps <- median(daily_activity$steps)

g1 <- ggplot(data = daily_activity, aes(x = steps)) + 
        geom_histogram(binwidth = 1000, alpha = 0.8, 
                       fill = "light blue") + 
        geom_vline(aes(xintercept = mean_steps, colour = "Mean daily steps")) +
        geom_vline(aes(xintercept = median_steps, colour = "Median daily steps")) +
        labs(x = "Dayly number of steps", y = "Count", 
             title = "Histogram of the total number of steps taken each day") +
        theme(legend.position = "bottom")
print(g1)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

Observe that there are 10 days during which no activity is recorded.

The mean and the median are

```r
print(mean_steps)
```

```
## [1] 9354.23
```

```r
print(median_steps)
```

```
## [1] 10395
```

## What is the average daily activity pattern?

In order to analyze the daily activity pattern, we compute first the average number of steps taken during each 5-minute interval (accross all day). The resulting time series is then plotted.


```r
int_activity <- activity %>% group_by(interval) %>% 
        summarise(avg_steps = mean(steps, na.rm = TRUE))

g2 <- ggplot(data = int_activity, aes(x = interval, y = avg_steps)) +
        geom_line(color = "light blue", size = 1) +
        labs(title = "Time series of the 5-minute interval average number of steps",
                     x = "Interval identifier", y = "Average number of steps taken")
print(g2)
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

The maximum numer of steps and the corresponding 5-minute interval are

```r
filter(int_activity, avg_steps == max(avg_steps))
```

```
## # A tibble: 1 × 2
##   interval avg_steps
##      <int>     <dbl>
## 1      835  206.1698
```

## Imputing missing values

As already mentioned, a considerable number of days/intervals are associated with missing values which may introduce bias into some calculations or summaries of the data. To circumvent this situation, we will now devise a simple strategy for filling in all of the missing values in the dataset.

The total number of missing values is

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

The strategy adopted in this work consists in replacing the missing values by the average number of steps of the corresponding interval.


```r
activityC <- activity %>% 
        transform(steps = ifelse(is.na(activity$steps), 
                                 int_activity$avg_steps[match(int_activity$interval,activity$interval)], 
                                 activity$steps
                                 ))

summary(activityC$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    0.00    0.00    0.00   37.38   27.00  806.00
```


Using the new dataset ```activityC```, we now make a histogram of the total number of steps taken each day,


```r
daily_activityC <- activityC %>% group_by(date) %>% 
        summarise(steps = sum(steps, na.rm = TRUE))

mean_steps_C <- mean(daily_activityC$steps)
median_steps_C <- median(daily_activityC$steps)

g3 <- ggplot(data = daily_activityC, aes(x = steps)) + 
        geom_histogram(binwidth = 1000, alpha = 0.8, 
                       fill = "light blue") + 
        geom_vline(aes(xintercept = mean_steps_C, colour = "Mean daily steps")) +
        geom_vline(aes(xintercept = median_steps_C, colour = "Median daily steps")) +
        labs(x = "Dayly number of steps", y = "Count", 
             title = "Histogram of the total number of steps taken each day (Cleaned dataset)") +
        theme(legend.position = "bottom")
print(g3)
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

As it can be observed on the previous Figure, the mean and median daily steps calculated with the *cleaned* dataset increase and, additionally, have the same values.

```r
print(mean_steps_C)
```

```
## [1] 10766.19
```

```r
print(median_steps_C)
```

```
## [1] 10766.19
```

Also, as a result of the adopted strategy, the number of days with an average daily steps of about 11000 drastically increases (close to a 50% increase).

## Are there differences in activity patterns between weekdays and weekends?

Unsing the *cleaned* dataset, we create a new ```Factor``` variable with two levels -- ```weekday``` and ```weekend``` indicating whether a given date is a weekday or weekend day. A panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
int_activityC <- activityC %>% mutate(daytype = ifelse(
        weekdays(date) %in% c("Saturday", "Sunday"),
        "Weekend", "Weekday")) %>% group_by(interval, daytype) %>%
        summarise(avg_steps = mean(steps))

g4 <- ggplot(data = int_activityC, aes(x = interval, y = avg_steps)) +
        geom_line(color = "light blue", size = 1) +
        facet_grid(daytype~.) +
        labs(title = "Time series of the 5-minute interval average number of steps (Cleaned dataset)",
                     x = "Interval identifier", y = "Average number of steps taken")
print(g4)
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

Finally, here are the mean number of steps taken by day type (i.e., ```Weekend``` or ```Weekday```):

```r
aggregate(avg_steps ~ daytype, int_activityC, mean)
```

```
##   daytype avg_steps
## 1 Weekday  35.61058
## 2 Weekend  42.36640
```

