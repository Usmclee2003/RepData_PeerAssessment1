Reproducible Research
================
Lee Pena
October 15, 2017

R Markdown
----------

This is my R markdown file that I am submitting for the Cousera course Reproducible Research

Bring into library necessary packages that will be used then read in the data set and convert date column to date format as it comes in as a factor

Also suppressing warnings and messages

``` r
# Turn off warnings and messages#
options(warn=-1)
suppressMessages(library(ggplot2))
suppressMessages(library(dplyr))

# read data in from csv file#
data <- read.csv(file = "activity.csv", header = TRUE, sep = ",")

#Convert date column from factor to a date#
data$date <- as.Date(data$date)
```

Look at data and structure to make sure data was transformed and data came in

``` r
str(data)
```

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

``` r
head(data)
```

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

What is mean total number of steps taken per day?
=================================================

For this part of the assignment, you can ignore the missing values in the dataset.
----------------------------------------------------------------------------------

Calculate the total number of steps taken per day If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day Calculate and report the mean and median of the total number of steps taken per day

``` r
#Sum number of steps by date#
stepsperday <- aggregate(steps~date, data,sum)
head(stepsperday)
```

    ##         date steps
    ## 1 2012-10-02   126
    ## 2 2012-10-03 11352
    ## 3 2012-10-04 12116
    ## 4 2012-10-05 13294
    ## 5 2012-10-06 15420
    ## 6 2012-10-07 11015

``` r
# Calculate and report Mean and Median values #
MEAN_stepsperday <- aggregate(steps~date, data,mean)
colnames(MEAN_stepsperday)<- c("Date", "Mean Steps per Day")
head(MEAN_stepsperday)
```

    ##         Date Mean Steps per Day
    ## 1 2012-10-02            0.43750
    ## 2 2012-10-03           39.41667
    ## 3 2012-10-04           42.06944
    ## 4 2012-10-05           46.15972
    ## 5 2012-10-06           53.54167
    ## 6 2012-10-07           38.24653

``` r
MEDIAN_stepsperday <- aggregate(steps~date, data,median)
colnames(MEDIAN_stepsperday)<- c("Date", "Median Steps per Day")
head(MEDIAN_stepsperday)
```

    ##         Date Median Steps per Day
    ## 1 2012-10-02                    0
    ## 2 2012-10-03                    0
    ## 3 2012-10-04                    0
    ## 4 2012-10-05                    0
    ## 5 2012-10-06                    0
    ## 6 2012-10-07                    0

``` r
# Create Histogram for number of steps per day
hist(stepsperday$steps, main = "Total steps per day", col = "lightblue", xlab = "Steps Taken in Day", ylab = "Number of Days Occurred")
```

![](RepRes_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-3-1.png)

What is the average daily activity pattern?
===========================================

Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis) Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

``` r
# remove na's from data set
data2<- na.omit(data)

#avg steps per 5 min interval
MEAN_stepsper_5_mins <- aggregate(steps~interval, data,median)
colnames(MEAN_stepsper_5_mins)<- c("Interval", "Mean_Steps")

# Make a times series plot

        #Timeseries5mins <- ts(MEAN_stepsper_5_mins$Mean_Steps, frequency = 105120 )
        #plot.ts(Timeseries5mins)  Could not get the frequency to look correct on x axis

TimeSeriesplot <- ggplot(MEAN_stepsper_5_mins, aes(Interval,Mean_Steps)) + geom_line() + xlab("5 min interval") + ylab("Mean Steps")
TimeSeriesplot
```

![](RepRes_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-4-1.png)

``` r
# Find 5 min interval that contains max value
Maxinterval<-MEAN_stepsper_5_mins[MEAN_stepsper_5_mins$Mean_Steps==max(MEAN_stepsper_5_mins$Mean_Steps),]
Maxinterval
```

    ##     Interval Mean_Steps
    ## 106      845         60

Imputing missing values
=======================

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs) Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc. Create a new dataset that is equal to the original dataset but with the missing data filled in. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

``` r
# Calculate and repot total number of missing values
missing_sum <- sum(is.na(data))
missing_sum
```

    ## [1] 2304

``` r
#Impute missing values with mean  of each interval
data3<-data %>%
  group_by(interval) %>% 
  mutate_all(funs(replace(., which(is.na(.)),
                           mean(., na.rm=TRUE))))
head(data3)
```

    ## # A tibble: 6 x 3
    ## # Groups:   interval [6]
    ##       steps       date interval
    ##       <dbl>     <date>    <int>
    ## 1 1.7169811 2012-10-01        0
    ## 2 0.3396226 2012-10-01        5
    ## 3 0.1320755 2012-10-01       10
    ## 4 0.1509434 2012-10-01       15
    ## 5 0.0754717 2012-10-01       20
    ## 6 2.0943396 2012-10-01       25

``` r
#Histogram of imputed data set
stepsperday3 <- aggregate(steps~date, data3,sum)
hist(stepsperday3$steps, main = "Total steps per day", col = "lightgreen", xlab = "Steps Taken in Day", ylab = "Number of Days Occurred")
```

![](RepRes_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-5-1.png)

``` r
#Report Mean and Median of imputed data
MEAN_stepsperday3 <- aggregate(steps~date, data3,mean)
colnames(MEAN_stepsperday3)<- c("Date", "Mean Steps per Day")
head(MEAN_stepsperday3)
```

    ##         Date Mean Steps per Day
    ## 1 2012-10-01           37.38260
    ## 2 2012-10-02            0.43750
    ## 3 2012-10-03           39.41667
    ## 4 2012-10-04           42.06944
    ## 5 2012-10-05           46.15972
    ## 6 2012-10-06           53.54167

``` r
MEDIAN_stepsperday3 <- aggregate(steps~date, data3,median)
colnames(MEDIAN_stepsperday3)<- c("Date", "Median Steps per Day")
head(MEDIAN_stepsperday3)
```

    ##         Date Median Steps per Day
    ## 1 2012-10-01             34.11321
    ## 2 2012-10-02              0.00000
    ## 3 2012-10-03              0.00000
    ## 4 2012-10-04              0.00000
    ## 5 2012-10-05              0.00000
    ## 6 2012-10-06              0.00000

Are there differences in activity patterns between weekdays and weekends?
=========================================================================

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.
--------------------------------------------------------------------------------------------------------------------------------

Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

``` r
#add column with classification of weekend or week day
data3$day<-ifelse(weekdays(data3$date) %in% c("Saturday","Sunday"),"Weekend", "Weekday")
head(data3)
```

    ## # A tibble: 6 x 4
    ## # Groups:   interval [6]
    ##       steps       date interval     day
    ##       <dbl>     <date>    <int>   <chr>
    ## 1 1.7169811 2012-10-01        0 Weekday
    ## 2 0.3396226 2012-10-01        5 Weekday
    ## 3 0.1320755 2012-10-01       10 Weekday
    ## 4 0.1509434 2012-10-01       15 Weekday
    ## 5 0.0754717 2012-10-01       20 Weekday
    ## 6 2.0943396 2012-10-01       25 Weekday

``` r
#dataWeekends<-data3[data3$day=='Weekend',]  ended up not needing this
#datadays<-data3[data3$day=='Weekday',]  or this
MEAN_stepsperday4 <- aggregate(steps~day+interval, data3,mean)
colnames(MEAN_stepsperday4)<- c("day","interval",'mean_steps')

#Create a Time Series multiplot
TimeSeriesplot2 <- ggplot(MEAN_stepsperday4, aes(interval,mean_steps)) + geom_line() + xlab("5 min interval")+ ylab("Mean Steps") +ggtitle("Mean Steps over Time")+facet_grid(day~.,labeller = label_parsed)
TimeSeriesplot2
```

![](RepRes_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-6-1.png)

``` r
#For some reason I could not get this to plot the data although the graph was created
#TimeSeriesplot3<- xyplot( MEAN_stepsperday4$mean_steps ~ MEAN_stepsperday4$interval | MEAN_stepsperday4$day,
          #               layout = c(1,2), type = "1", xlab = "5 min Interval", ylab = "Mean Steps")
```
