Reproducible Research: Peer Assessment 1
======================================================

<br>

## Loading and preprocessing the data


```r
library(ggplot2)                                # load libraries used in this assignment
library(dplyr)

data <- read.csv("activity.csv")                # read the data
data <- tbl_df(data)                            # convert to dplyr data frame
```

<br>



## What is mean total number of steps taken per day?

First, plot a histogram of the data


```r
dailySum <- group_by(data, date)                          # group data by date
dailySum <- summarize(dailySum, Steps = sum(steps))       # sum by date

h <- ggplot(dailySum, aes(x = Steps)) + 
        geom_histogram(fill = "skyblue", color = "darkgray") +
        labs(title = "Histogram of Total Daily Steps\n") +
        scale_y_discrete(breaks = seq(0, 10, by = 1))

print(h)
```

![plot of chunk histogram](figure/histogram-1.png) 

Next, calculate the Mean and Median of Total Daily Steps


```r
sMean <- round(mean(dailySum$Steps, na.rm = T), 2)        # round to 2 decimal places
sMedian <- median(dailySum$Steps, na.rm = T)
```

#### Mean = 10766.19  
#### Median = 10765

<br>



## What is the average daily activity pattern?

First, create a time series plot of the 5-minute intervals and the average number of steps
taken (averaged across all days)


```r
avgIntv <- group_by(data, interval)                             # group data by interval
avgIntv <- summarize(avgIntv, Steps = mean(steps, na.rm = T))   # average by interval

tsPlot <- ggplot(avgIntv, aes(x = interval, y = Steps)) + 
        geom_line(color = "blue") +
        labs(title = "Plot of Average Steps Taken per Interval (Across all Days)\n") +
        scale_x_discrete(breaks = seq(0, 2400, by = 100))

print(tsPlot)
```

![plot of chunk time series plot](figure/time series plot-1.png) 

Next, determine which 5-minute interval contains the maximum average number of steps


```r
maxSteps <- max(avgIntv$Steps)                          # finds max value in Steps column
maxIntv <- with(avgIntv, interval[Steps==maxSteps])     # finds which interval it is
```

#### Interval 835 has the maximum average number of steps = 206.17

<br>



## Imputing missing values

First, determine the total number of missing values in the dataset (i.e. total number of
rows with NAs)


```r
totalNAs <- is.na(data$steps)           # creates a logical vector of which rows have NAs
totalNAs <- sum(as.numeric(totalNAs))   # convert TRUEs to 1 and sum
```

#### Total number of missing values is 2304

Next, my strategy for filling in missing values in the dataset is to replace each NA value with the mean (calculated above) for that 5-minute interval it is found in.


```r
# convert the data back to a normal data frame
data2 <- data.frame(data)
# find the row numbers in the dataset that are NAs (i.e. missing)
rowNums <- as.integer(rownames(data2[is.na(data2$steps), ]))
# get the intervals the missing data belongs to
vIntervals <- data2[rowNums, "interval"]

# loop on each row that is missing data and replace it with the mean value corresponding
# to its 5-minute interval
for (i in 1:length(rowNums)) {
        data2[rowNums[i], "steps"] <- avgIntv[avgIntv$interval==vIntervals[i], "Steps"]
}
```

Now, with the missing values imputed, create a histogram of the of the total number of steps taken each day and then calculate the Mean and Median.


```r
dailySum2 <- group_by(data2, date)                          # group data by date
dailySum2 <- summarize(dailySum2, Steps = sum(steps))       # sum by date

h2 <- ggplot(dailySum2, aes(x = Steps)) + 
        geom_histogram(fill = "skyblue", color = "darkgray") +
        labs(title = "Histogram of Total Daily Steps (Imputed Data)\n") +
        scale_y_discrete(breaks = seq(0, 12, by = 1))

print(h2)
```

![plot of chunk imputed histogram](figure/imputed histogram-1.png) 

```r
sMean2 <- mean(dailySum2$Steps)
sMedian2 <- median(dailySum2$Steps)
```

#### Mean = 10766.19  
#### Median = 10766.19

It appears my strategy for imputing the data only increased the number of days the data set was near the mean, which the median almost proves. It went from 4 days observed in its bucket to 12.

<br>

## Are there differences in activity patterns between weekdays and weekends?

First, I'm going to add a new column that designates whether it is weekday or weekend and then average the imputed data in each 5-minute interval across the common day type. Then I'm going to plot two time series charts that compare weekday steps to weekend steps.


```r
data2 <- tbl_df(data2)

data2 <- mutate(data2, day = weekdays(as.Date(date)))
data2$day <- gsub("Saturday", "weekend", data2$day)
data2$day <- gsub("Sunday", "weekend", data2$day)
data2$day <- gsub("Monday", "weekday", data2$day)
data2$day <- gsub("Tuesday", "weekday", data2$day)
data2$day <- gsub("Wednesday", "weekday", data2$day)
data2$day <- gsub("Thursday", "weekday", data2$day)
data2$day <- gsub("Friday", "weekday", data2$day)

avgIntv2 <- group_by(data2, interval, day)              # group data by day by interval
avgIntv2 <- summarize(avgIntv2, Steps = mean(steps))    # average by interval

tsPlot2 <- ggplot(avgIntv2, aes(x = interval, y = Steps)) + 
        geom_line(color = "blue") +
        facet_wrap(~ day, ncol=1) +
        labs(title = "Plot of Average Steps Taken per Interval (Weekday vs. Weekend)\n") +
        scale_x_discrete(breaks = seq(0, 2400, by = 100))

print(tsPlot2)
```

![plot of chunk panel plot](figure/panel plot-1.png) 

On weekdays there appears to be more steps taken in the morning and evening, whereas weekends show more steps during the middle of the day. 


