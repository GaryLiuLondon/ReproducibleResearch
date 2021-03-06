Reproducible Research - Assignment 1
=====================================

##Loading and preprocessing the data

1.Load the data (i.e. read.csv())


```r
data<-read.csv(file="activity.csv", na.strings="NA",header=T, colClasses=c("numeric","character","numeric"))
```
2.Process/transform the data (if necessary) into a format suitable for your analysis


```r
data$date<-as.Date(data$date)
```
##Question 1: What is mean toal number of steps taken per day?
1.Make a histogram of the total number of steps taken each day


```r
require(plyr)
total<-ddply(data,c("date"),summarize,sum=sum(steps, na.rm=T))
plot(total$sum,type="h")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

2.Calculate and report the mean and median total number of steps taken per day


```r
meantotal<-mean(total$sum)
mediantotal<-median(total$sum)
```
So the mean of total number of steps taken per day is 9354.2295 and median is 1.0395 &times; 10<sup>4</sup>

##Question 2: What is the average daily activity pattern?

1.Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
mininterval<-ddply(data, c("interval"),summarize,mean=mean(steps,na.rm=T))
plot(mininterval$mean~mininterval$interval,type="l")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

2.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
mininterval[mininterval$mean==max(mininterval$mean),c("interval")]
```

```
## [1] 835
```

##Imputing missing values
1.Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
sumNA<-sum(is.na(data$steps))
```
Total number of missing values in the dataset is 2304

2.Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
imputedval<-ddply(data,c("interval"),summarize, filler=median(steps,na.rm=T))
# imputedval[is.nan(imputedval$filler),"filler"]<-0
# imputedval
```
3.Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
data.out<-data
for(sourceinterval in imputedval$interval)
    {
        data.out[data.out$interval==sourceinterval & is.na(data.out$steps),"steps"]<-imputedval[imputedval$interval==sourceinterval,"filler"]
    }
```
4.Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
plotdata<-ddply(data.out,c("date"),summarize, allsteps=sum(steps))
plot(plotdata$allsteps,type="h")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 



```r
# plotdata
meanallsteps<-mean(plotdata$allsteps)
```

The mean total number of steps taken in each day is9503.8689

```r
medianallsteps<-median(plotdata$allsteps)
```

The median number of steps taken in each day is 1.0395 &times; 10<sup>4</sup>

###Question 3: Are there differences in activity patterns between weekdays and weekends?

1.Create a new factor variable in the dataset with two levels ? ?weekday? and ?weekend? indicating whether a given date is a weekday or weekend day.


```r
data.out$wk<-ifelse(weekdays(data.out$date) %in% c("Saturday","Sunday"),2,1)
data.out$wk<-factor(data.out$wk)
levels(data.out$wk)<-c("weekday","weekend")
```
2.Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was creating using simulated data:


```r
require(ggplot2)
plotdata<-ddply(data.out,c("interval","wk"),summarize,meansteps=mean(steps))
# plotdata
p<-ggplot(plotdata, aes(x=interval, y=meansteps)) + geom_line()+facet_grid(wk ~ .)
p
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14.png) 
