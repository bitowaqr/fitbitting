---
title: "R Notebook"
output: html_notebook
---

## fitbitting 

#### ANALYZING SLEEPING PATTERNS COLLECTED BY FITBIT

__Setup fitbit API__: 

  Go to teramonagi github page
  on the fitbitr package
   -->     https://github.com/teramonagi/fitbitr
   And follow the instructions to set up the fitbit API
  
  
#### THEN: Get auth
```
  # install.packages("devtools")
  # devtools::install_github("teramonagi/fitbitr")
  library(fitbitr)
  library(httr)

  FITBIT_KEY    <- "your_key"
  FITBIT_SECRET <- "your_secret"
  token <- oauth_token(key = FITBIT_KEY, secret = FITBIT_SECRET)
```

#### Load sleeping pattern analysis function:
  Thanks to mmparker :  https://gist.github.com/mmparker/54f1fd05e62671f80dfb4b855231e6ee

```
    analyze.my.sleep = function(start_date="2017-01-01",
                                end_date="2018-06-14",
                                limit = c("Monday","Tuesday" ,  "Thursday" ), # NULL
                                contrast = c("Wednesday" ,"Friday" , "Saturday",  "Sunday"  ), # NULL
                                nrows = 2,
                                smooth_span = 0.25,
                                bar_spacing = 0.1
                                ){


      require(lubridate)
      require(ggplot2)
      options(stringsAsFactors = FALSE)


        sleep.start = get_sleep_time_series(token, c("startTime"), base_date=start_date, end_date=end_date)
        times = paste(sleep.start$dateTime," ",sleep.start$value,":01 MET",sep="")
        times[nchar(times)<20] = NA
        times = as.POSIXlt(times)

        sleep.duration = get_sleep_time_series(token, c("minutesAsleep"), base_date=start_date, end_date=end_date)
        duration = sleep.duration$value

        sleepytimes <- data.frame(start_sleep = times)
        sleepytimes$sleep_minutes <- as.numeric(duration)
        sleepytimes$sleep_hours <- floor(sleepytimes$sleep_minutes/60)


        sleepytimes$end_sleep <- with(sleepytimes,
                                      start_sleep + duration(minutes = sleep_minutes)
        )


        ### REMOVE MISSINGS
        sleepytimes = sleepytimes[!is.na(sleepytimes$start_sleep),]


        # LIMITS/CONTRASTS:
        if(!is.null(limit)){

          if(sum(c("Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday","Monday") %in% limit)>0){
            indexing = format(sleepytimes$start_sleep,"%A")
            index = indexing %in% limit
          }
          if(sum(c("June","July","August","September","October",
                   "November","December","January","February",
                   "March","April","May") %in% limit)>0){
            indexing = format(sleepytimes$start_sleep,"%A")
            index = indexing %in% limit
          }

          sleepytimes.2 = sleepytimes[index,]
          sleepytimes.2$type = paste(limit,collapse = "+")

        if(!is.null(contrast)){

          if(sum(c("Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday","Monday") %in% contrast)>0){
            indexing = format(sleepytimes$start_sleep,"%A")
            index = indexing %in% contrast
          }
          if(sum(c("June","July","August","September","October",
                   "November","December","January","February",
                   "March","April","May") %in% contrast)>0){
            indexing = format(sleepytimes$start_sleep,"%A")
            index = indexing %in% contrast
          }

          sleepytimes.3 = sleepytimes[index,]
          sleepytimes.3$type = paste(contrast,collapse = "+")

          sleepytimes.2 = rbind(sleepytimes.2,sleepytimes.3)
        }

        sleepytimes = sleepytimes.2

        } else {sleepytimes$type = " "}
        # sleepytimes = sleepytimes[100:300,]
        # Now, to plot... you're totally right that crossing zero is going to be a
        # problem if we treat it like a datetime. So I think the easiest thing to do
        # subtract midnight from the sleep and wake times...

        # First, I'm going to try to associate every sleep period with a particular date,
        # even if sleep didn't start until after midnight. Rule of thumb: if sleep
        # starts before 12pm on Tuesday, it's counted for Monday night
        sleepytimes$night_of <- with(sleepytimes,

                                     # The ifelse() function returns integers instead of Dates for some reason,
                                     # so gotta convert back to Dates
                                     as.Date(

                                       # Sleep started before 12pm? If yes, use previous date
                                       ifelse(hour(start_sleep) > 12,
                                              yes = date(start_sleep),
                                              no = date(start_sleep) - 1),

                                       origin = "1970-01-01")

        )


        # Now I'll use the difftime() function to calculate hours until/hours since
        # midnight for all of the sleep start times - these will usually be positive:
        sleepytimes$start_sleep_diff <- with(sleepytimes,
                                             as.numeric(
                                               difftime(time1 =as.POSIXct(paste((night_of + 1), "00:00:00")),
                                                        time2 = start_sleep,
                                                        units = "hours")
                                             )
        )


        # And again for all of the wake times - these will usually be negative:
        sleepytimes$end_sleep_diff <- with(sleepytimes,
                                           as.numeric(
                                             difftime(time1 = as.POSIXct(paste((night_of + 1), "00:00:00")),
                                                      time2 = end_sleep,
                                                      units = "hours")
                                           )
        )

        label_hours <- function(x) {

          require(lubridate)

          # Start with midnight, then subtract the "time to midnight" variables - 
          # just reversing what we did before, basically, then formatting
          # to show just the hours (%I) and AM/PM indicator (%p)
          format(as.POSIXct(paste(Sys.Date(), "00:00:00")) - hours(x), "%I:00 %p")

        }




        plot.durations = 
        ggplot(sleepytimes) +
          geom_hline(yintercept = c(2,-8),size=2) +
          geom_point(aes(x = night_of,y= start_sleep_diff),col="orange") +
          geom_point(aes(x = night_of,y= end_sleep_diff),col="red") +
          geom_rect(aes(xmin = night_of + bar_spacing, 
                        xmax = night_of + (1 - bar_spacing),
                        ymin = start_sleep_diff,
                        ymax = end_sleep_diff),
                    fill = "#2b8cbe") +
          geom_smooth(aes(x = night_of, y = start_sleep_diff), 
                      method = loess, method.args = list(span = smooth_span),
                      se = FALSE, color = "#045a8d") +

          geom_smooth(aes(x = night_of, y = end_sleep_diff), 
                      method = loess, method.args = list(span = smooth_span),
                      se = FALSE, color = "#045a8d") +
          labs(x = "Date",
               y = "Time",
               main = "Sleep and Wake Times") +
          scale_y_reverse(labels = label_hours, breaks = seq(-24,24,by=3),
                          minor_breaks = seq(-24,24,by=1)) +
          theme_bw() +
          coord_flip() +
          theme(axis.text.x = element_text(angle = 45, hjust = 1)) 




        if ( !is.null(contrast)){
          plot.durations = 
            plot.durations + 
            facet_wrap(~type,
                       nrow = nrows) 
          }

        # regression analysis
        sleepytimes$day = format(sleepytimes$start_sleep,"%A")
        lm.fun.day = lm(sleep_minutes/60 ~ +1+day , data = sleepytimes)
        sum.lm.day = summary(lm.fun.day)

        sleepytimes$month = format(sleepytimes$start_sleep,"%B")
        lm.fun.month = lm(sleep_minutes/60 ~ +1+month , data = sleepytimes)
        sum.lm.month = summary(lm.fun.month)

        # boxplot
        box = ggplot(sleepytimes) +
          geom_boxplot(aes(x=day, y=sleep_minutes/60)) +
          ylab("Hours of sleep")

        # sleep over time
        sleep.per.weekday.plot =
        ggplot(sleepytimes)  +
           geom_line(aes(x=start_sleep,y=sleep_minutes/60)) +
          geom_smooth(aes(x = start_sleep, y = sleep_minutes/60), 
                      method = loess, method.args = list(span = smooth_span),
                      se = T, color = "#045a8d") +
          ylab("Hours of sleep")



        # histo sleep hours over time
        histo = ggplot(sleepytimes)  +
          geom_histogram(aes(sleep_minutes/60),bins=50,fill="lightblue",col="blue") +
          xlab("Hours of sleep") +
          ylab("Frequency") +
          geom_vline(xintercept = mean(sleepytimes$sleep_minutes/60)) +
          scale_x_continuous(breaks=1:12) +
          theme_minimal() 



        output = list(plot.durations = plot.durations,
                      sleep.per.weekday.plot = sleep.per.weekday.plot,
                      histo = histo,
                      box = box,
                      plot.starts = plot.starts,
                      lm.fun.day = lm.fun.day,
                      lm.fun.month = lm.fun.month,
                      dta = sleepytimes)
        return(output)
        }
```

#### FINALLY: RUN IT

```
test.set = analyze.my.sleep(start_date="2017-01-01",
                            end_date="2018-06-14",
                            limit = c("Monday","Tuesday" ,  "Thursday" ), # NULL
                            contrast = c("Wednesday" ,"Friday" , "Saturday",  "Sunday"  ), # NULL
                            nrows = 2,
                            smooth_span = 0.25,
                            bar_spacing = 0.1)

# AND HAVE A LOOK
test.set$plot.durations

test.set$box
test.set$lm.fun.day

test.set$sleep.per.weekday.plot
test.set$lm.fun.month

test.set$histo
```


