---
title: "Reproducible Research - Week 2 Project"
output: 
  html_document:
    keep_md: true
---
## Loading and preprocessing the data
```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", destfile = "week2.zip")
unzip("week2.zip")
week2data<-read.csv("activity.csv")
```




## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.
Calculate the total number of steps taken per day
```{r}
subdata<-week2data[,1:2]
complete<-subdata[complete.cases(subdata),]
aggdata<-aggregate(complete$steps, by=list(complete$date), FUN = sum)
names(aggdata)<-c("date", "steps")
aggdata$date<-as.Date(as.character(aggdata$date))
mean(aggdata$steps)
#10766.19
median(aggdata$steps)
#10765
```

If you do not understand the difference between a histogram and a barplot, research the difference between them. 
Make a histogram of the total number of steps taken each day 
```{r}
hist(aggdata$steps, breaks = 30, xlab="steps/day")
```




## What is the average daily activity pattern?

Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
```{r}
intervaldata<-week2data[complete.cases(week2data),-2]
avgdata<-aggregate(intervaldata$steps, by=list(intervaldata$interval), mean)
names(avgdata)<-c("Interval", "avg steps")
plot(avgdata$Interval, avgdata$`avg steps`, type = "l", xlab="time",ylab="avg steps")
```

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
```{r}
max(avgdata$`avg steps`)
#206.1698
max_interval<-avgdata[avgdata$`avg steps`>206.169,]
max_interval
```




## Imputing missing values

*Note that there are a number of days/intervals where there are missing values (coded as NA).*
*The presence of missing days may introduce bias into some calculations or summaries of the data.*

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
```{r}
miss<-is.na(week2data)
sum(miss)
```

Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. 
For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
```{r}
library(dplyr)
FillData<-function(data){
  replace(data, is.na(data), mean(data, na.rm=T))
}
```

Create a new dataset that is equal to the original dataset but with the missing data filled in.
```{r}
filledData<-(week2data %>% group_by(interval) %>% mutate(steps = FillData(steps)))
sum(complete.cases(filledData))
sum(complete.cases(week2data))
```

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.
```{r}
aggFill<-aggregate(filledData$steps, by=list(filledData$date), sum)
names(aggFill)<-c("date", "steps")
hist(aggFill$steps, breaks = 30, xlab="steps/day")
mean(aggFill$steps)
median(aggFill$steps)
```

Do these values differ from the estimates from the first part of the assignment?
```{r}
par(mfcol=c(1,2))
hist(aggdata$steps, breaks = 30, xlab="steps/day", main="Original Data")
hist(aggFill$steps, breaks = 30, xlab="steps/day", main="Filled Data")
```

What is the impact of imputing missing data on the estimates of the total daily number of steps?

**From the comparison, we can see that the highest count of the new version data is larger than the one we have with NAs. The means of each dataset are same. The medians of each dataset are slightly different.**




## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.
```{r}
library(dplyr)
filledData$date<-as.POSIXct(filledData$date)
weekfilledData<-mutate(filledData, weekday=weekdays(date))
weekdaydata<-mutate(weekfilledData, day=ifelse(weekday=="Saturday" | weekday=="Sunday", "weekend", "weekday"))
weekdaydata$day<-as.factor(weekdaydata$day)
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, 
averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look 
like using simulated data.
```{r}
wkday<-filter(weekdaydata, day=="weekday")
wkend<-filter(weekdaydata, day=="weekend")
avgwkday<-aggregate(wkday$steps, by=list(wkday$interval), mean)
names(avgwkday)<-c("Interval", "avg steps")
avgwkend<-aggregate(wkend$steps, by=list(wkend$interval), mean)
names(avgwkend)<-c("Interval", "avg steps")
par(mfcol=c(2,1))
plot(avgwkday$Interval, avgwkday$`avg steps`, type = "l", xlab="",ylab="Number of steps", main = "Weekday")
plot(avgwkend$Interval, avgwkend$`avg steps`, type = "l", xlab="Interval",ylab="Number of steps", main="Weekend")
```

