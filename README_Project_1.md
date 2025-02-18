# Reproducible Research - Week 2 - Project 1 

### 1. Loading and preparing the data

#### Loading libraries
```{r}
library(ggplot2)
library(dplyr)
```

#### Loading the data (file 'activity.csv' in the working directory)
```{r}
activity <- read.csv("activity.csv")
```
#### Processing for analysis
```{r}
# printing out the first 20 rows
head(activity,20)
```

```{r}
# Aggregating 
activity$date2<-as.Date(as.character(activity$date), '%Y-%m-%d')
activity_ag <- aggregate(steps~date2, data=activity, FUN=sum,na.rm=TRUE)
activity_ag<-activity_ag[order(activity_ag$date2),]
```

### What is mean total number of steps taken per day?

#### Plotting

```{r}
ggplot(activity_ag) + geom_col(aes(x=date2,y=steps,group=date2,fill=steps)) +
  xlab("Dates Range")+
  ylab("Total Steps per Day")+
  ggtitle("Total Steps Per Day")+
  theme(plot.margin=unit(c(0,0,0,0),"mm"))
```
![Plot1](https://github.com/msrcos3s/RepData_PeerAssessment1/blob/master/Plots%20Prints/Figure1.png)

### Mean and median number of steps taken each day
```{r}
activity_int <- aggregate(steps ~ interval, data=activity, FUN=mean)
print(paste0("Mean steps per day: ", steps_mean<-mean(activity_ag$steps)))
print(paste0("Median steps per day: ", steps_median<-median(activity_ag$steps)))
```

### Time series plot of the average number of steps taken
```{r}
activity_interval_ag <- aggregate(steps ~ interval, activity, mean, rm.na=T)
plot(x = activity_interval_ag$interval,y = activity_interval_ag$steps,
        type = "l",lwd=3, col="magenta",
        main = "5 Minute Intervals \n Average Number of Steps per Interval",
        xlab = "5 Minute Intervals Mean",
        ylab = "Average Number of Steps")
```
![Plot2](https://github.com/msrcos3s/RepData_PeerAssessment1/blob/master/Plots%20Prints/Figure2.png)

### The 5-minute interval that, on average, contains the maximum number of steps
```{r}
print(paste0("The 5-minute interval that contains the maximum number of steps: ",
activity_interval_ag$interval[which.max(activity_interval_ag$steps)]))
max_steps <- which.max(activity_interval_ag$steps)
activity_interval_ag[max_steps, ]
```

### Code to describe and show a strategy for imputing missing data
```{r}
sum(is.na(activity))
#creating a new dataset for adding data to missing values
activity_merge <- activity

#Making a function to return steps mean from time interval
step_inteveral <- function(int){
  activity_interval_ag$steps[activity_interval_ag$interval==int]
}

#Replacing NAs with means
activity_merge$steps[is.na(activity_merge$steps)]<- round(as.numeric(lapply(activity_merge$interval[is.na(activity_merge$steps)],step_inteveral), digits = 0))

#Aggregating both datasets
activity_merge_ag <- aggregate(steps~date2, data=activity_merge, FUN=sum,na.rm=TRUE)
activity_merge_ag<-activity_merge_ag[order(activity_merge_ag$date2),]
```
### Histogram of the total number of steps taken each day after missing values are imputed
```{r}
ggplot(activity_merge_ag) + geom_col(aes(x=date2,y=steps,group=date2,fill=steps)) +
  xlab("Dates Range (Imputed Data)")+
  ylab("Total Steps per Day")+
  ggtitle("Total Steps Per Day")+
  theme(plot.margin=unit(c(0,0,0,0),"mm"))
```
![Plot3](https://github.com/msrcos3s/RepData_PeerAssessment1/blob/master/Plots%20Prints/Figure3.png)

#### Mean and median total number of steps taken per day WITHOUT filling in the missing values
```{r}
summary (activity_ag$steps)
```
#### Mean and median total number of steps taken per day WITH filling in the missing values
```{r}
summary (activity_merge_ag$steps)
```

### Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends
```{r}

activity_interval_merge_ag <- aggregate(steps ~ interval, activity_merge, mean, rm.na=T)

activity_weekends <- activity[weekdays(as.Date(activity$date)) %in% c("Saturday", "Sunday"), ]
activity_weekdays <- activity[weekdays(as.Date(activity$date)) %in% c("Monday","Tuesday","Wednesday","Thursday","Friday"), ]

activity_int_weekend_ag <- aggregate(steps ~ interval, activity_weekends, mean, rm.na=T)
activity_int_weekday_ag <- aggregate(steps ~ interval, activity_weekdays, mean, rm.na=T)
par(mfrow = c(2, 1))
plot(x = activity_int_weekend_ag$interval,y = activity_int_weekend_ag$steps,
        type = "l",lwd=3, col="red", main = "5-Minute Intervals \n Weekends", xlab = "5-Minute Intervals Averages (Mean)", ylab = "Steps", ylim=c(0,200),xlim=c(0,2500))

plot(x = activity_int_weekday_ag$interval,y = activity_int_weekday_ag$steps,
        type = "l",lwd=3, col="orange", main = "5-Minute Intervals \n Weekdays", xlab = "5-Minute Intervals Averages (Mean)", ylab = "Steps", ylim=c(0,200),xlim=c(0,2500))
```
![Plot4](https://github.com/msrcos3s/RepData_PeerAssessment1/blob/master/Plots%20Prints/Figure4.png)
