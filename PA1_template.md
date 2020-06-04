This document was created to answer the questions defined in the
Reproducible Research: Peer Assessment 1 assignment.

Loading and preprocessing the data
----------------------------------

First, the dplyr package was loaded before reading the csv file as a
dataframe. Lubridate can also be used to convert the strings in the date
column to dates that R recognises.

    library(dplyr)
    library(lubridate)
    data <- data.frame(read.csv("repdata_data_activity/activity.csv"))
    data <- data %>% mutate(date = ymd(date))

What is mean total number of steps taken per day?
-------------------------------------------------

The data can be grouped by the date, then the total number of steps per
day can be found (ignoring missing values).

    perDay <- data %>% group_by(date) %>% summarise(total = sum(steps, na.rm = TRUE))

A histogram of the total number of steps per day can be plotted using
the ggplot2 package. Since histograms use frequency/count instead of a
given y-value like a bar chart, a vector can be created which repeats
each date by its total number of steps.

    library(ggplot2)
    histPerDay <- data.frame(rep(perDay$date, perDay$total)) # dataframe required for ggplot2
    names(histPerDay) <- "date" # rename default name to something useful
    ggplot(data = histPerDay, aes(date)) +
        geom_histogram(binwidth = 1) +
        labs(title = "Total Number of Steps per Day", x = "Date", y = "Total Number of Steps") +
        theme_minimal() +
        theme(plot.title = element_text(hjust = 0.5)) # center graph title

![](PA1_template_files/figure-markdown_strict/total%20steps%20histogram-1.png)

Using the perDay dataframe calculated above, the mean and the mediam
total number of steps taken per day can be found (again ignoring missing
values).

    mean(perDay$total, na.rm = TRUE)

    ## [1] 9354.23

    median(perDay$total, na.rm = TRUE)

    ## [1] 10395

What is the average daily activity pattern?
-------------------------------------------

The original data can be grouped by the 5-minute interval for every day.

    perInterval <- data %>% group_by(interval) %>% summarise(mean = mean(steps, na.rm = TRUE))

This can then be plotted in a line chart to show the mean number of
steps across all days for each 5-minute interval.

    ggplot(data = perInterval, aes(x = interval, y = mean)) +
        geom_line() +
        labs(title = "Average Number of Steps across All Days per 5-Minute Interval", x = "Interval", y = "Mean Number of Steps") +
        theme_minimal() +
        theme(plot.title = element_text(hjust = 0.5))

![](PA1_template_files/figure-markdown_strict/mean%20steps%20per%20interval-1.png)

The interval with the maximum mean number of steps across all days can
be found by finding the row number and then subsetting the interval
column.

    perInterval$interval[which(perInterval$mean == max(perInterval$mean))]

    ## [1] 835

Imputing missing values
-----------------------

There are a number of rows in the original data that are missing values
in the steps column, and this number can be found using the code below.

    sum(is.na(data$steps))

    ## [1] 2304

To fill in these missing values, the mean number of steps across all
days, calculated above, will be used. The interval at each NA is first
found, then used to find the replacement value.

    naRows <- data.frame(which(is.na(data$steps))) # create new dataframe with row numbers for NA values
    names(naRows) <- "row" # rename default colymn to something meaningful
    naRows <- naRows %>% mutate(interval = data$interval[row]) # create new interval column that uses row number to get interval for NA value
    naRows <- naRows %>% mutate(replacement = perInterval$mean[match(interval, perInterval$interval)]) # create new replacement column that matches the interval value from naRows to the interval value in perInterval, returning the mean for that interval across all days

A new dataframe can be made by copying the original data, and then
replacing all values of NA with the mean for that interval across all
days.

    replacedData <- data # copy original dataframe
    replacedData$steps[naRows$row] <- naRows$replacement # replace the NA values identified by the row number using the replacement value found above

The total number of steps per day can be plotted on a histogram using
the same code as above, but without ignoring NA values.

    replacedPerDay <- replacedData %>% group_by(date) %>% summarise(total = sum(steps))
    replacedHistPerDay <- data.frame(rep(replacedPerDay$date, replacedPerDay$total))
    names(replacedHistPerDay) <- "date"
    ggplot(data = replacedHistPerDay, aes(date)) +
        geom_histogram(binwidth = 1) +
        labs(title = "Total Number of Steps per Day", x = "Date", y = "Total Number of Steps") +
        theme_minimal() +
        theme(plot.title = element_text(hjust = 0.5))

![](PA1_template_files/figure-markdown_strict/total%20steps%20histogram%20with%20replaced%20NAs-1.png)

Using the same code as before, the mean and the median can also be found
with the replaced NA values.

    mean(replacedPerDay$total)

    ## [1] 10766.19

    median(replacedPerDay$total)

    ## [1] 10766.19

Both the mean and the median were found to increase when the NA values
were replaced. In fact, the mean and the median were found to be the
same with the replaced NA values.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

The data with replaced NA values can be grouped based on whether it was
on a weekday or a weekend. To do so, the day of the week can be
determined. If it is a Saturday or a Sunday then it will be classed as a
weekend, and all other days will be classed as a weekday.

    replacedData <- replacedData %>% mutate(weekday = ifelse(weekdays(date) %in% c("Saturday", "Sunday"), "weekend", "weekday")) # uses weekdays() function to determine day of the week, sets "Saturday" and "Sunday" values to "weekend" and all other values to "weekday" in new weekday column

The mean steps for each interval for all weekdays can be stored in a
dataframe, with the mean steps for each interval for all weekends stored
in another.

    wdayPerInterval <- replacedData %>% filter(weekday == "weekday") %>% group_by(interval) %>% summarise(mean = mean(steps))
    wendPerInterval <- replacedData %>% filter(weekday == "weekend") %>% group_by(interval) %>% summarise(mean = mean(steps))

These can then be displayed in a panel plot of line graphs. The
gridExtra package needs to be loaded to allow a grid plot using ggplot2.

    library(gridExtra)
    wdayPlot <- ggplot(wdayPerInterval, aes(x = interval, y = mean)) +
        geom_line() +
        labs(title = "weekday", x = "Interval", y = "Mean Number of Steps") +
        theme_minimal() +
        theme(plot.title = element_text(hjust = 0.5))
    wendPlot <- ggplot(wendPerInterval, aes(x = interval, y = mean)) +
        geom_line() +
        labs(title = "weekend", x = "Interval", y = "Mean Number of Steps") +
        theme_minimal() +
        theme(plot.title = element_text(hjust = 0.5))
    grid.arrange(wendPlot, wdayPlot, nrow = 2)

![](PA1_template_files/figure-markdown_strict/panel%20plot-1.png)
