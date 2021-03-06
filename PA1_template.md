# Reproducible Research: Peer Assessment 1

## Assignment  
This assignment will be described in multiple parts. You will need to write a report that answers the questions detailed below. Ultimately, you will need to complete the entire assignment in a single R markdown document that can be processed by knitr and be transformed into an HTML file.  
Throughout your report make sure you always include the code that you used to generate the output you present. When writing code chunks in the R markdown document, always use echo = TRUE so that someone else will be able to read the code. This assignment will be evaluated via peer assessment so it is essential that your peer evaluators be able to review the code for your analysis.  
For the plotting aspects of this assignment, feel free to use any plotting system in R (i.e., base, lattice, ggplot2).  
Fork/clone the GitHub repository created for this assignment. You will submit this assignment by pushing your completed files into your forked repository on GitHub. The assignment submission will consist of the URL to your GitHub repository and the SHA-1 commit ID for your repository state.  
NOTE: The GitHub repository also contains the dataset for the assignment so you do not have to download the data separately.  

## Step1: Loading and preprocessing the data

Show any code that is needed to  
Load the data (i.e. read.csv())

```r
setwd("D:\\GIT\\DataScience\\RepData_PeerAssessment1")
activity_temp <- read.csv("activity.csv", stringsAsFactors=FALSE)
```

Process/transform the data (if necessary) into a format suitable for your analysis

```r
# Change date format
activity_temp$date <- as.POSIXct(activity_temp$date, format="%Y-%m-%d")
# Weekdays
activity_temp <- data.frame(date=activity_temp$date, 
                           weekday=tolower(weekdays(activity_temp$date)), 
                           steps=activity_temp$steps, 
                           interval=activity_temp$interval)
# Day type
activity_temp <- cbind(activity_temp, 
                      daytype=ifelse(activity_temp$weekday == "saturday" | 
                                     activity_temp$weekday == "sunday", "weekend", 
                                     "weekday"))
# Create data.frame
activity <- data.frame(date=activity_temp$date, 
                       weekday=activity_temp$weekday, 
                       daytype=activity_temp$daytype, 
                       interval=activity_temp$interval,
                       steps=activity_temp$steps)

# Delete temp df
rm(activity_temp)
```

Head it:

```r
head(activity)
```

```
##         date weekday daytype interval steps
## 1 2012-10-01  monday weekday        0    NA
## 2 2012-10-01  monday weekday        5    NA
## 3 2012-10-01  monday weekday       10    NA
## 4 2012-10-01  monday weekday       15    NA
## 5 2012-10-01  monday weekday       20    NA
## 6 2012-10-01  monday weekday       25    NA
```




## Step2: What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.

1.Calculate the total number of steps taken per day

```r
total_steps <- aggregate(activity$steps, by=list(activity$date), FUN=sum, na.rm=TRUE)
names(total_steps) <- c("date", "total")
head(total_steps)
```

```
##         date total
## 1 2012-10-01     0
## 2 2012-10-02   126
## 3 2012-10-03 11352
## 4 2012-10-04 12116
## 5 2012-10-05 13294
## 6 2012-10-06 15420
```
2.If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

```r
hist(total_steps$total, 
     breaks=seq(from=0, to=25000, by=2500),
     col="darkgreen", 
     xlab="Total number of steps", 
     ylim=c(0, 20), 
     main="Histogram - total number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->
3.Calculate and report the mean and median of the total number of steps taken per day

```r
total.steps <- tapply(activity$steps, activity$date, FUN = sum, na.rm = TRUE)
```

```r
mean(total.steps)
```

```
## [1] 9354.23
```

```r
median(total.steps)
```

```
## [1] 10395
```

## Step3: What is the average daily activity pattern?

```r
mean_data <- aggregate(activity$steps, 
                       by=list(activity$interval), 
                       FUN=mean, 
                       na.rm=TRUE)
names(mean_data) <- c("interval", "mean")
head(mean_data)
```

```
##   interval      mean
## 1        0 1.7169811
## 2        5 0.3396226
## 3       10 0.1320755
## 4       15 0.1509434
## 5       20 0.0754717
## 6       25 2.0943396
```
Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)  

```r
plot(mean_data$interval, 
     mean_data$mean, 
     type="l", 
     col="blue", 
     lwd=2, 
     xlab="Interval [minutes]", 
     ylab="Average number of steps", 
     main="Average number of steps across all days")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->
Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
max_pos <- which(mean_data$mean == max(mean_data$mean))
max_interval <- mean_data[max_pos, 1]
max_interval
```

```
## [1] 835
```

## Step4: Imputing missing values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
rm(max_interval)
NA_count <- sum(is.na(activity$steps))
NA_count
```

```
## [1] 2304
```
2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
rm(NA_count)
na_pos <- which(is.na(activity$steps))
mean_vec <- rep(mean(activity$steps, na.rm=TRUE), times=length(na_pos))
str(mean_vec)
```

```
##  num [1:2304] 37.4 37.4 37.4 37.4 37.4 ...
```
3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
activity[na_pos, "steps"] <- mean_vec
rm(mean_vec, na_pos)
head(activity)
```

```
##         date weekday daytype interval   steps
## 1 2012-10-01  monday weekday        0 37.3826
## 2 2012-10-01  monday weekday        5 37.3826
## 3 2012-10-01  monday weekday       10 37.3826
## 4 2012-10-01  monday weekday       15 37.3826
## 5 2012-10-01  monday weekday       20 37.3826
## 6 2012-10-01  monday weekday       25 37.3826
```
4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
sum_data <- aggregate(activity$steps, by=list(activity$date), FUN=sum)
names(sum_data) <- c("date", "total")
hist(sum_data$total, 
     breaks=seq(from=0, to=25000, by=2500),
     col="darkgreen", 
     xlab="Total number of steps", 
     ylim=c(0, 30), 
     main="Histogram - total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png)<!-- -->


## Are there differences in activity patterns between weekdays and weekends?
For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.  
1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
# It is done in step 1 (preparation)
head(activity)
```

```
##         date weekday daytype interval   steps
## 1 2012-10-01  monday weekday        0 37.3826
## 2 2012-10-01  monday weekday        5 37.3826
## 3 2012-10-01  monday weekday       10 37.3826
## 4 2012-10-01  monday weekday       15 37.3826
## 5 2012-10-01  monday weekday       20 37.3826
## 6 2012-10-01  monday weekday       25 37.3826
```
2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
rm(sum_data)
library(lattice)
mean_data <- aggregate(activity$steps, 
                       by=list(activity$daytype, 
                               activity$weekday, activity$interval), mean)
names(mean_data) <- c("daytype", "weekday", "interval", "mean")
xyplot(mean ~ interval | daytype, mean_data, 
       type="l", 
       lwd=1, 
       xlab="Interval", 
       ylab="Number of steps", 
       layout=c(1,2))
```

![](PA1_template_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

