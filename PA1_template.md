Exploratory analysis of activity data
=====================================

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012 and
include the number of steps taken in 5 minute intervals each day.

The data for this assignment can be downloaded
[here](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip).

The variables included in this dataset are:

-   steps: Number of steps taking in a 5-minute interval (missing values
    are coded as NA)
-   date: The date on which the measurement was taken in YYYY-MM-DD
    format
-   interval: Identifier for the 5-minute interval in which measurement
    was taken

The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this dataset.

We begin the analysis by loading the dataset and then using the dplyr
package to convert it into a suitable format.

### Task 1: Load dataset

    #step 1: Load the data
    download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip",
                  "./activity.zip")
    unzip("./activity.zip")
    file_input<-read.csv("./activity.csv",sep=",")

We then proceed to check for missing values in the dataset. The aim is
to compute the Total,mean and median number of steps taken per day.
Since there are missing values in this dataset, we consider two
scenarios:

1.  Remove the NA values in the dataset and then compute the Total, mean
    and median number of steps taken per day
2.  Fill in the NA values by defining our own stratergy

Scenario 1:
-----------

#### Sub step 1: Process dataset.

Remove missing(NA) values in dataset.

    #Let us omit NA values
    file_input_na_rm<-na.omit(file_input)

    #Let us use dplyr package to convert it to a suitable format to be used

    library(dplyr)
    data<-tbl_df(file_input_na_rm)

### Task 2: Compute the mean total number of steps taken per day

#### Sub task 1:

calculate the total number of steps taken per day

In order to do this we first group the table by date and then add the
steps pertaining to a particular date.

    data<-data%>%
            group_by(date)%>%
            summarize(total=sum(steps))

#### Sub task 2:

Histogram of total number of steps taken per day

    library(ggplot2)
    ggplot(data=data,aes(total,fill="pink"))+
            geom_histogram()+
            labs(
                 title= "Histogram of total number of steps taken per day on data with omitted NA",
                 x="Total number of steps taken per day",
                 y="Frequency")+
            theme_bw()

![](PA1_template_files/figure-markdown_strict/Plot1-1.png)

#### Sub task 3:

Calculate and report the mean and median of number of steps per day

    data<-tbl_df(file_input_na_rm)%>%
            group_by(date)%>% #group by date and then compute the mean and median of steps belonging to the date
            summarize(Mean=mean(steps),Median=median(steps))%>%# now we take the mean and median of steps per day
            summarize(Grand_mean=mean(Mean),Grand_median=median(Median))

    head(data)

    ## # A tibble: 1 x 2
    ##   Grand_mean Grand_median
    ##        <dbl>        <dbl>
    ## 1       37.4            0

Before we perform the computations in scenario 2 and then compare the
results generated in the two cases, we need to first devise a stratergy
to fill in the missing values.

#### Stratergy:

Compute the average steps taken in each unique interval and fill in the
missing steps by mapping the intervals that correspond to the missing
interval with the unique interval.

### Task 3: Compute average daily pattern

#### Sub task 1:

Prepare a time series plot of the 5-minute interval (x-axis) and the
average number of steps taken, averaged across all days (y-axis)

    data_2<-tbl_df(file_input_na_rm)%>%
            group_by(interval)%>%
            summarize(average=mean(steps),total=sum(steps))# use this data in next step to fill in missing steps
    with(data_2,
         plot(x=interval,
              y=average,
              type="l",
              col="blue",
              main = "Average number of steps taken averaged across all days",
              xlab="Interval",
              ylab="Average number of steps taken per day"))

![](PA1_template_files/figure-markdown_strict/Plot%202-1.png)

##### Sub task 2:

Compute 5-minute interval, on average across all the days in the dataset
that contains the maximum number of steps

    data_2%>%arrange(desc(total))

    ## # A tibble: 288 x 3
    ##    interval average total
    ##       <int>   <dbl> <int>
    ##  1      835    206. 10927
    ##  2      840    196. 10384
    ##  3      850    183.  9720
    ##  4      845    180.  9517
    ##  5      830    177.  9397
    ##  6      820    171.  9071
    ##  7      855    167.  8852
    ##  8      815    158.  8349
    ##  9      825    155.  8236
    ## 10      900    143.  7603
    ## # ... with 278 more rows

We can see that the interval 835 contains the maximum number of steps.

Scenario 2:
===========

Imputing missing values
-----------------------

#### Sub task 1:

Calculate and report the total number of missing values in the dataset
(i.e. the total number of rows with NA)

    #Get raw data( With NA values)
    dat<-tbl_df(file_input)

    #filter records with NA values
    dt_NA<-dat%>%filter(is.na(steps))

    #Imputation: Fill in steps with NA values with mean values of steps computed in previous step unique to an interval
    for(i in 1:nrow(dt_NA))
    {
            dt_NA$steps[i]<-data_2$average[data_2$interval==dt_NA$interval] # map intervals to obtain the average value
    }

    #Filter records without NA values
    dt_NT_NA<-dat%>%filter(!is.na(steps))

    #Join records with NA values post imputation with recordds without NA values.
    dat<-rbind(dt_NA,dt_NT_NA)

    #Now check if computation was sucessful.Are there NA values?
    length(which(is.na(dat$steps)))

    ## [1] 0

WE see that the computation was sucessful and there are no more missing
values in the datset. Repeat sub tasks 2 and 3 in previous scenario

#### Sub task 2:

Histogram of total number of steps taken per day.

    new_dat<-dat%>%group_by(date)%>%summarize(total=sum(steps))
    ggplot(data=new_dat,aes(total,fill="pink"))+geom_histogram()  +
            labs(x="Total number of steps taken per day",
                 y="Frequency",
                 title= "Histogram of total number of steps taken per day on imputed data"
                 )+
            theme_bw()

![](PA1_template_files/figure-markdown_strict/Plot%203-1.png)

### Comprison: Scenario 2 vs Sceanrio 1

    data_imputed_NA<-dat%>%group_by(date)%>%summarize(total=sum(steps))
    data_imputed_NA$label<-c(rep("Imputed NA values",dim(data_imputed_NA)[1]))

    data_omitted_NA<-tbl_df(file_input_na_rm)%>%group_by(date)%>%summarize(total=sum(steps))
    data_omitted_NA$label<-c(rep("Data without NA values",dim(data_omitted_NA)[1]))

    data_compare<-bind_rows(data_omitted_NA,data_imputed_NA)

    library(ggplot2)
    ggplot(data=data_compare,aes(total,fill=label,alpha=0.2))+
            geom_histogram()+
            labs(title="Compare distribution of total number of steps taken per day between when NA values were imputed and omitted",
                 x="Total number of steps taken per day",
                 y="Frequency")+
            theme_bw()

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-7-1.png)

The blue bars indicate records containing NA values that have been
imputed. We can see that there are a lot of these bars and if omiited
could result in loss of information. \#\#\#\# Sub task 3:

Compute mean and median of number of steps per day

    data<-dat%>%
            group_by(date)%>%
            summarize(Mean=mean(steps),Median=median(steps))%>%# compute mean and median of steps for each date
            summarize(Grand_mean=mean(Mean),Grand_median=median(Median))# compute the mean and median number of steps taken per day

    head(data)

    ## # A tibble: 1 x 2
    ##   Grand_mean Grand_median
    ##        <dbl>        <dbl>
    ## 1       32.7            0

### Task 4: Check for differences between weekdays and weekends

#### Sub task 1:

Create a new factor variable in the dataset with two levels - "weekday"
and "weekend" indicating whether a given date is a weekday or weekend
day.

    #Differences between weekdays and weekends
    data_4<-dat%>%
            mutate(day=ifelse(weekdays(as.Date(date))%in%
                                      c("Saturday","Sunday"),"Weekend","Weekday"))%>%
            group_by(day,interval)%>%summarize(average=mean(steps))

#### Subtask 2:

Make a panel plot containing a time series plot (i.e.type="l") of the
5-minute interval (x-axis) and the average number of steps taken,
averaged across all weekday days or weekend days (y-axis).

    library(lattice)
    xyplot(average~interval|day,data=data_4,layout=c(1,2),type="l",ylab="Average number of steps taken per day",xlab="Interval")

![](PA1_template_files/figure-markdown_strict/Plot%205-1.png)
