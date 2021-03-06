---
title: "Reproducible Research Assessment 1"
author: "Florian Refardt"
date: "Thursday, March 12, 2015"
output: html_document
---

## Loading and preprocessing the data

Show any code that is needed to

Load the data (i.e. read.csv())


```{r}

activity <- read.csv("activity.csv", stringsAsFactors = FALSE)

```


Process/transform the data (if necessary) into a format suitable for your analysis

```{r}

# Aggregation of the total steps taken per day
stepsPerDay <- aggregate(activity$steps, by = list(activity$date), FUN=sum, na.rm = TRUE)
names(stepsPerDay) <- c("date", "stepsPerDay")

# Aggregation of the mean steps taken per 5 min. interval
avgStepsPerDay <- aggregate(activity$steps, by = list(activity$interval), FUN = mean, na.rm = TRUE)
names(avgStepsPerDay) <- c("interval", "avgSteps")



```


What is mean total number of steps taken per day?

```{r}

mean(stepsPerDay$stepsPerDay, na.rm = TRUE)


```


For this part of the assignment, you can ignore the missing values in the dataset.

Calculate the total number of steps taken per day

```{r}

sum(stepsPerDay$stepsPerDay, na.rm = TRUE)

```


If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day


```{r}

hist(x = stepsPerDay$stepsPerDay, breaks = 10, ylim = c(0, 20))

```


Calculate and report the mean and median of the total number of steps taken per day

```{r}

median(stepsPerDay$stepsPerDay, na.rm = TRUE)
mean(stepsPerDay$stepsPerDay, na.rm = TRUE)

```


What is the average daily activity pattern?

Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```{r}

plot(avgStepsPerDay, type = "l")

```


Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```{r}

avgStepsPerDay[which.max(avgStepsPerDay$avgSteps),1]

```


Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```{r}

sum(is.na(activity$steps))

```


Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```{r}

naIndex <- which(is.na(activity$steps))

meansPerInterval <- rep(mean(activity$steps, na.rm=TRUE), times=length(naIndex))

activity[naIndex, "steps"] <- meansPerInterval

```


Create a new dataset that is equal to the original dataset but with the missing data filled in.

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```{r}

# Aggregation of the total steps taken per day
stepsPerDay <- aggregate(activity$steps, by = list(activity$date), FUN=sum, na.rm = TRUE)
names(stepsPerDay) <- c("date", "stepsPerDay")

# Aggregation of the mean steps taken per 5 min. interval
avgStepsPerDay <- aggregate(activity$steps, by = list(activity$interval), FUN = mean, na.rm = TRUE)
names(avgStepsPerDay) <- c("interval", "avgSteps")

```

```{r}

hist(x = stepsPerDay$stepsPerDay, breaks = 10, ylim = c(0, 30))

```


Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```{r}

# Dataframe with pre-calculated weekday and weekend index
date <- as.POSIXct(activity$date, format= "%Y-%m-%d")
weekday = weekdays.Date(date)
isWeekend = ifelse(weekday == "Samstag" | weekday == "Sonntag", 1, 0)

activityFrame <- data.frame(
                        date = date,
                        interval = activity$interval,
                        steps = activity$steps,
                        weekday = weekday,
                        isWeekend = isWeekend        
                )

```


Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```{r}

library(ggplot2)
require(gridExtra)

activityFrameMean <- aggregate(activityFrame$steps, by = list(activityFrame$date, activityFrame$isWeekend, activityFrame$interval), mean)
names(activityFrameMean) <- c("date", "isWeekend", "avgStepsPerDay")

onlyWeekdays <- subset(activityFrameMean, activityFrameMean$isWeekend == 0)
onlyWeekends <- subset(activityFrameMean, activityFrameMean$isWeekend == 1)

plotWeekdays <- qplot(date, avgStepsPerDay, data = onlyWeekdays, geom = "line", main="Weekdays")
plotWeekends <- qplot(date, avgStepsPerDay, data = onlyWeekends, geom = "line", main="Weekends")

grid.arrange(plotWeekdays, plotWeekends, ncol=1)

```
