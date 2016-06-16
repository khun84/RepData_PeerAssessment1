Introduction
------------

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement -- a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

Data
----

The data for this assignment can be downloaded from the course web site:

-   **Dataset**: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) \[52K\]

The variables included in this dataset are:

-   **steps**: Number of steps taking in a 5-minute interval (missing values are coded as NA)

-   **date**: The date on which the measurement was taken in YYYY-MM-DD format

-   **interval**: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

Assignment
----------

This assignment will be described in multiple parts. You will need to write a report that answers the questions detailed below. Ultimately, you will need to complete the entire assignment in a **single R markdown** document that can be processed by knitr and be transformed into an HTML file.

Throughout your report make sure you always include the code that you used to generate the output you present. When writing code chunks in the R markdown document, always use echo = TRUE so that someone else will be able to read the code. **This assignment will be evaluated via peer assessment so it is essential that your peer evaluators be able to review the code for your analysis.**

For the plotting aspects of this assignment, feel free to use any plotting system in R (i.e., base, lattice, ggplot2)

Fork/clone the [GitHub repository created for this assignment](http://github.com/rdpeng/RepData_PeerAssessment1). You will submit this assignment by pushing your completed files into your forked repository on GitHub. The assignment submission will consist of the URL to your GitHub repository and the SHA-1 commit ID for your repository state.

NOTE: The GitHub repository also contains the dataset for the assignment so you do not have to download the data separately.

Loading and preprocessing the data
----------------------------------

Show any code that is needed to

1.  Load the data (i.e. `read.csv()`)

``` r
# Load the dataset from the csv file
activity <- read.csv('/Users/Daniel/COURSERA DATA SCIENCE/reproducible research/Assignment 1/RepData_PeerAssessment1/activity.csv', stringsAsFactors = F, na.strings = 'NA')
```

1.  Process/transform the data (if necessary) into a format suitable for your analysis

``` r
# Change to data table
library(data.table)
activity <- data.table(activity)

str(activity)
```

    ## Classes 'data.table' and 'data.frame':   17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
    ##  - attr(*, ".internal.selfref")=<externalptr>

``` r
# Change the date type to date format
library(lubridate)
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following objects are masked from 'package:data.table':
    ## 
    ##     hour, mday, month, quarter, wday, week, yday, year

    ## The following object is masked from 'package:base':
    ## 
    ##     date

``` r
activity[, date:= ymd(date)]
```

What is mean total number of steps taken per day?
-------------------------------------------------

For this part of the assignment, you can ignore the missing values in the dataset.

1.  Make a histogram of the total number of steps taken each day

We perform an aggregation by summing the steps by day and follow by plotting aggregated steps in histogram.

``` r
# Calculate the total number of steps per day
total.steps.day <- activity[, .(total = sum(steps, na.rm = T)), date]
```

Next we look at the **summary** of the aggregated steps

``` r
summary(total.steps.day$total)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##       0    6778   10400    9354   12810   21190

Plot the aggregated steps in histogram using the default binning parameter.

``` r
# Plot the historgram
hist(total.steps.day$total, xlab = 'Total number of steps by day', main = 'Histogram of total number of steps by day', col = 'grey')
```

![](PA1_template_files/figure-markdown_github/plot%20histogram-1.png)<!-- -->

1.  Calculate and report the **mean** and **median** total number of steps taken per day

``` r
# Load library
library(dplyr)
```

``` r
# Calculate mean and median of total number of steps per day, round to 2 digit
mean.total.steps.day <- mean(total.steps.day$total, na.rm = T) %>% round(2)
med.total.steps.day <- median(total.steps.day$total, na.rm = T) %>% round(2)
```

The **mean** and **median** of the total of steps per day are **9354.23** and **10395**.

What is the average daily activity pattern?
-------------------------------------------

1.  Make a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

We aggregate the data by averaging the steps across all days by 5-minute interval

``` r
# Calculate the average of steps across all days by 5-min interval
mean.total.steps.interval <- activity[, .(mean = mean(steps, na.rm = T)), interval]
```

Plot the aggregated steps (y-axis) vs 5-minute interval (x-axis) in line chart.

``` r
plot(mean.total.steps.interval$interval, mean.total.steps.interval$mean, type = 'l', xlab = '5-min interval', ylab = 'Average of steps across all days', main = 'Average of steps across all days ~ 5-min interval')
```

![](PA1_template_files/figure-markdown_github/plot%20line%20chart-1.png)<!-- -->

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
-------------------------------------------------------------------------------------------------------------

``` r
# Identify the position index of the maximum steps
pos = which.max(mean.total.steps.interval$mean)

# Identify the interval with maximum steps
max.step.interval <- mean.total.steps.interval$interval[pos]
```

The 5-minute interval that contains the maximum number of steps is **`835`**.

Inputing the missing values
---------------------------

Note that there are a number of days/intervals where there are missing values (coded as `NA`). The presence of missing days may introduce bias into some calculations or summaries of the data.

1.  Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with `NA`’s)

``` r
# Calculate number of NA in each column
na.count <- sapply(activity, function(x){is.na(x) %>% sum()})

# Show the result of NA calculation
na.count
```

    ##    steps     date interval 
    ##     2304        0        0

We see that only column 'steps' has missing values and the number of missing values is **`2304`**

1.  Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

We populate the calculate the mean column for 5-minute interval and append it to the `activity` data table.

``` r
# populate the mean (group by 5 min interval) column
activity[, mean.na:=mean(steps, na.rm = T), interval]
```

1.  Create a new dataset that is equal to the original dataset but with the missing data filled in.

``` r
# create new dataset by replace NA values in column steps with mean for 5 min interval
activity.new <- activity[, .(steps, interval, date, mean.na)]
activity.new[, steps:=ifelse(is.na(steps), mean.na, steps)]

# check if there is still any NA in the new dataset
is.na(activity.new$steps) %>% sum()
```

    ## [1] 0

1.  Make a histogram of the total number of steps taken each day and Calculate and report the **mean** and **median** total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

We first compute the sum of steps taken by each day.

``` r
# Calculate sum of steps taken by each day
steps.sum <- activity.new[, .(sum = sum(steps)), date]

# show the first few records of steps.sum
head(steps.sum)
```

    ##          date      sum
    ## 1: 2012-10-01 10766.19
    ## 2: 2012-10-02   126.00
    ## 3: 2012-10-03 11352.00
    ## 4: 2012-10-04 12116.00
    ## 5: 2012-10-05 13294.00
    ## 6: 2012-10-06 15420.00

Plot the histogram of the sum of steps taken by each day.

``` r
# Plot sum of steps taken by each day using the default binning parameter
hist(steps.sum$sum, xlab = 'Sum of steps taken by each day', main = 'Histogram of Sum of Steps Taken by Each Day (NA replaced by mean)', col = 'grey')
```

![](PA1_template_files/figure-markdown_github/plot%20historgram%20of%20sum%20of%20steps-1.png)<!-- -->

Calculate the **mean** and **median** total number of steps taken per day.

``` r
# Calculate the mean and median of steps taken by each day
steps.mean <- mean(steps.sum$sum)
steps.med <- median(steps.sum$sum)
```

The **mean** is **`10766.19`** and the **median** is **`10766.19`**. These values are sligher larger than the first part of the assignment. The imputation process had actually introduced more data point to the dataset. As all these imputed values are positive, the mean and median had actually increased though the increment is marginal.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

For this part the `weekdays()` function may be of some help here. Use the dataset with the filled-in missing values for this part.

1.  Create a new factor variable in the dataset with two levels - “weekdays” and “weekend” indicating whether a given date is a weekday or weekend day.

Create a new column to describe the day of the date.

``` r
# create a new column for the name of the day
activity.new[, day:=weekdays(date)]
head(activity.new)
```

    ##        steps interval       date   mean.na    day
    ## 1: 1.7169811        0 2012-10-01 1.7169811 Monday
    ## 2: 0.3396226        5 2012-10-01 0.3396226 Monday
    ## 3: 0.1320755       10 2012-10-01 0.1320755 Monday
    ## 4: 0.1509434       15 2012-10-01 0.1509434 Monday
    ## 5: 0.0754717       20 2012-10-01 0.0754717 Monday
    ## 6: 2.0943396       25 2012-10-01 2.0943396 Monday

Next create another column to indicate 'weekday' or 'weekend'.

``` r
#create new column to indicate weekday or weekend
activity.new[, daytype:=ifelse(day == 'Sunday' | day == 'Saturday', 'weekend', 'weekday')]

# change the new column data type ot factor
activity.new[, daytype:= as.factor(daytype)]

head(activity.new)
```

    ##        steps interval       date   mean.na    day daytype
    ## 1: 1.7169811        0 2012-10-01 1.7169811 Monday weekday
    ## 2: 0.3396226        5 2012-10-01 0.3396226 Monday weekday
    ## 3: 0.1320755       10 2012-10-01 0.1320755 Monday weekday
    ## 4: 0.1509434       15 2012-10-01 0.1509434 Monday weekday
    ## 5: 0.0754717       20 2012-10-01 0.0754717 Monday weekday
    ## 6: 2.0943396       25 2012-10-01 2.0943396 Monday weekday

1.  Make a panel plot containing a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

In order to plot the graph, we have to calculate the average of steps grouped by `daytype` and `interval`.

``` r
# Calculate average of steps grouped by interval and daytype
stepsAverage <- activity.new[, .(steps = mean(steps)), .(daytype, interval)]
head(stepsAverage)
```

    ##    daytype interval      steps
    ## 1: weekday        0 2.25115304
    ## 2: weekday        5 0.44528302
    ## 3: weekday       10 0.17316562
    ## 4: weekday       15 0.19790356
    ## 5: weekday       20 0.09895178
    ## 6: weekday       25 1.59035639

Next we make the panel plot.

``` r
library(ggplot2)
# Make panel plot
qplot(interval, steps, data = stepsAverage, facets = daytype~., geom = 'line', xlab = 'Interval', ylab = 'Number of Steps', main = 'Mean of Steps vs 5-min Interval by Daytype')
```

![](PA1_template_files/figure-markdown_github/make%20panel%20plot-1.png)<!-- -->
