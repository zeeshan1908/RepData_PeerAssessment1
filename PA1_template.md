Load data
---------

    data<-read.csv("C:/Users/Zeeshan/Desktop/coursera/data1/activity.csv")
    head(data)

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

    library(ggplot2)
    library(mice)

    ## Warning: package 'mice' was built under R version 3.3.2

    ## Loading required package: Rcpp

    ## mice 2.25 2015-11-09

What is mean total number of steps taken per day?
-------------------------------------------------

removing NA from data frame

    data.wo.na <- data[complete.cases(data),]
    head(data.wo.na)

    ##     steps       date interval
    ## 289     0 2012-10-02        0
    ## 290     0 2012-10-02        5
    ## 291     0 2012-10-02       10
    ## 292     0 2012-10-02       15
    ## 293     0 2012-10-02       20
    ## 294     0 2012-10-02       25

taking sum of no. of steps taken per day

    date.step<-aggregate(data.wo.na[, 1], list(data.wo.na$date), sum)
    colnames(date.step)<-c("date","steps")

1.  plotting histogram of date and no of steps

<!-- -->

     ggplot(date.step,aes(x=date, y=steps)) + geom_bar(stat = "identity") + theme(axis.text.x = element_text(angle = 90, hjust = 1))

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-4-1.png)

1.  Calculate and report the mean and median total number of steps taken
    per day

<!-- -->

    Mean <- mean(date.step$steps)
    Mean

    ## [1] 10766.19

    Median <- median(date.step$steps)
    Median

    ## [1] 10765

What is the average daily activity pattern?
-------------------------------------------

finding average no of steps taken per interval across days

    averageStepsPerTimeBlock<-aggregate(data$steps, list(data$interval), mean, na.rm=T)
    colnames(averageStepsPerTimeBlock)<-c("interval","meansteps")
    head(averageStepsPerTimeBlock)

    ##   interval meansteps
    ## 1        0 1.7169811
    ## 2        5 0.3396226
    ## 3       10 0.1320755
    ## 4       15 0.1509434
    ## 5       20 0.0754717
    ## 6       25 2.0943396

1.  Make a time series plot

<!-- -->

    ggplot(data=averageStepsPerTimeBlock, aes(x=interval, y=meansteps)) +geom_line() + xlab("5-minute interval") +     ylab("average number of steps taken")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-7-1.png)

1.  Which 5-minute interval, on average across all the days in the
    dataset, contains the maximum number of steps?

<!-- -->

    mostSteps <- averageStepsPerTimeBlock[which.max(averageStepsPerTimeBlock$meansteps),]

most steps

    mostSteps

    ##     interval meansteps
    ## 104      835  206.1698

Imputing missing values
-----------------------

1.  Calculate and report the total number of missing values in the
    dataset

<!-- -->

    numMissingValues <- length(which(is.na(data$steps)))

Number of missing values

    numMissingValues

    ## [1] 2304

1.  Devise a strategy for filling in all of the missing values in
    the dataset.

-   using MICE package to impute NA by predictive mean matching (pmm)

1.  Create a new dataset that is equal to the original dataset but with
    the missing data filled in.

<!-- -->

    data1 <- data
    tempdata <- mice(data1, m=5, maxit=50, meth='pmm', seed=500)
    dataimputed <- complete(tempdata,1)

1.  Make a histogram of the total number of steps taken each day

<!-- -->

     ggplot(dataimputed,aes(x=date, y=steps)) + geom_bar(stat = "identity") + theme(axis.text.x = element_text(angle = 90, hjust = 1))

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-13-1.png)

... and Calculate and report the mean and median total number of steps
taken per day

    meansteps<-mean(dataimputed$steps)
    meansteps

    ## [1] 36.11071

    mediansteps<-median(dataimputed$steps)
    mediansteps

    ## [1] 0

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

1.  Create a new factor variable in the dataset with two levels -
    "weekday" and "weekend" indicating whether a given date is a weekday
    or weekend day.

<!-- -->

    dataimputed$datetype <-  ifelse(as.POSIXlt(dataimputed$date)$wday %in% c(0,6), 'weekend', 'weekday')
    head(dataimputed)

    ##   steps       date interval datetype
    ## 1     0 2012-10-01        0  weekday
    ## 2     0 2012-10-01        5  weekday
    ## 3     0 2012-10-01       10  weekday
    ## 4     0 2012-10-01       15  weekday
    ## 5     0 2012-10-01       20  weekday
    ## 6     0 2012-10-01       25  weekday

1.  Make a panel plot containing a time series plot

<!-- -->

    averagedActivityDataImputed <- aggregate(steps ~ interval + datetype, data=dataimputed, mean)
    ggplot(averagedActivityDataImputed, aes(interval, steps)) +    geom_line() + facet_grid(datetype ~ .) + xlab("5-minute interval") + ylab("avarage number of steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-16-1.png)
