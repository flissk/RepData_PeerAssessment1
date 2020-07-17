Info on dataset
---------------

The variables included in this dataset are:

-   `steps`: Number of steps taking in a 5-minute interval (missing
    values are coded as `NA`)
-   `date`: The date on which the measurement was taken in YYYY-MM-DD
    format
-   `interval`: Identifier for the 5-minute interval in which
    measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this dataset.

Loading relevant packages
-------------------------

    library(tidyverse)
    library(ggplot2)
    library(timeDate)

Loading and preprocessing the data
----------------------------------

    filename <- "repdata_data_activity.zip"

    #unzip file

    if (!file.exists("repdata_data_activity")) { 
        unzip(filename) 
    }

    #load data into R 

    data <- read_csv("activity.csv")

What is the mean total number of steps taken per day?
-----------------------------------------------------

The total number of steps taken per day is stored in the new dataframe
`temp`, and depicted in the histogram `hist_totalsteps`. The following
code returns `temp` and `hist_totalsteps`.

    # dataframe `temp` creates new column `daily_steps` indicating total number of steps taken per day 
    temp <- data %>%
      group_by(date) %>%
      summarize(daily_steps = sum(steps))

    hist_totalsteps <- hist(temp$daily_steps, xlab = "Steps taken per day", main = "Histogram of steps taken per day", col = "cadetblue3", ylim = c(0, 30), breaks = seq(0,25000, by = 2500))

![](rep_research_proj1_files/figure-markdown_strict/unnamed-chunk-3-1.png)

The mean number of steps taken each day is 10766.19.

    mean(temp$daily_steps, na.rm = TRUE) 

    ## [1] 10766.19

The median number of steps taken each day is 10765.

    median(temp$daily_steps, na.rm = TRUE) 

    ## [1] 10765

What is the average daily activity pattern?
-------------------------------------------

The following code returns a time series plot `steps_over_time`, which
depicts the average number of steps taken per day.

    #create new dataframe `temp3` with column `interval_steps` indicating the average number of steps per interval. 
    temp3 <- data %>%
      group_by(interval) %>%
      drop_na(steps) %>% #remove missing values 
      summarize(interval_steps = mean(steps))

    #plot time series graph
    steps_over_time <- ggplot(temp3, aes(x = interval, y = interval_steps), na.rm = TRUE, type = "1") +
      geom_line() +
      labs(
        title="Average Daily Activity",
        y="Steps taken",
        x="Time"
      )

    print(steps_over_time)

![](rep_research_proj1_files/figure-markdown_strict/unnamed-chunk-6-1.png)

From the plot, the average number of steps taken tends to be higher in
the middle part of the day.

    #subset temp3 to identify interval with maximum number of steps. 
    max_interval <- subset(temp3, temp3$interval_steps ==  max(temp3$interval_steps))
    print(max_interval)

    ## # A tibble: 1 x 2
    ##   interval interval_steps
    ##      <dbl>          <dbl>
    ## 1      835           206.

On average, the 835th interval contains the maximum number of steps.
Code:

Imputing missing values
-----------------------

    summary(data) #returns 2304 NA values 

    ##      steps             date               interval     
    ##  Min.   :  0.00   Min.   :2012-10-01   Min.   :   0.0  
    ##  1st Qu.:  0.00   1st Qu.:2012-10-16   1st Qu.: 588.8  
    ##  Median :  0.00   Median :2012-10-31   Median :1177.5  
    ##  Mean   : 37.38   Mean   :2012-10-31   Mean   :1177.5  
    ##  3rd Qu.: 12.00   3rd Qu.:2012-11-15   3rd Qu.:1766.2  
    ##  Max.   :806.00   Max.   :2012-11-30   Max.   :2355.0  
    ##  NA's   :2304

There are 2,304 missing values in the dataset (i.e. coded as`NA`)

One strategy for filling in all missing values in the dataset involves
replacing each missing value with the mean for that particular
five-minute interval.

    step_values_to_impute <- temp3$interval_steps[match(data$interval, temp3$interval)]

From here, we can create a new dataset that is equal to the original
dataset but with the missing data filled in.

    data_imputed <- data %>%
      mutate(steps = ifelse(is.na(data$steps), yes = step_values_to_impute, no = data$steps))

We can also create a histogram of the total number of steps taken each
day after missing values are imputed.

    #create new dataframe `data_imputed_2` with column `daily_steps` indicating the total daily steps taken. 
    data_imputed_2 <- data_imputed %>%
      group_by(date) %>%
      summarize(daily_steps = sum(steps))

    hist(data_imputed_2$daily_steps, main = "Total no. of steps taken per day", xlab = "Steps taken per day", col = "coral3", ylim = c(0, 40))

![](rep_research_proj1_files/figure-markdown_strict/unnamed-chunk-11-1.png)

After imputing missing values, we can further calculate the mean and
median total number of steps taken per day.

    mean(data_imputed_2$daily_steps)

    ## [1] 10766.19

    median(data_imputed_2$daily_steps)

    ## [1] 10766.19

Both the mean and median total number of steps taken per day is
10766.19, after imputing missing values. This is equivalent to the
estimated mean from the first part of the assignment, but larger than
the previously estimated median of 10765. This suggests that imputing
missing values will affect the estimated median but not the mean.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

First, we create a new factor variable in the dataset with two levels –
“weekday” and “weekend” indicating whether a given date is a weekday or
weekend day.

    #create a vector of weekdays

    data_week <- data_imputed %>%
      mutate(week_day = isWeekday(date)) %>%
      mutate(week_day = as.factor(week_day)) 

    levels(data_week$week_day)[1] <- "weekend"
    levels(data_week$week_day)[2] <- "weekday"

Following which, we can make a panel plot containing a time series plot
of the 5-minute interval (x-axis) and the average number of steps taken,
averaged across all weekday days or weekend days (y-axis).

    steps_model <- aggregate(formula = steps~interval + week_day, data = data_week, FUN = mean, na.rm = TRUE)

    steps_panel <- ggplot(steps_model, aes(x = interval, y = steps, color = week_day)) +
      facet_wrap(~week_day, nrow = 2, ncol = 1) +
      geom_line() +
      labs(
        title="Average Daily Activity",
        y="Steps taken",
        x="Time",
        color = "Type of day"
      )
      
    print(steps_panel)

![](rep_research_proj1_files/figure-markdown_strict/unnamed-chunk-15-1.png)

Finally, from the plot, we can conclude that there is little difference
in activity patterns between weekends and weekdays. In both cases, the
number of steps taken tends to peak in the mornings, between 0730 to
1000, and are at their lowest before 0500 as well as after 2230.
