---
title: "R Notebook"
output: html_notebook
---

## fitbitting 

#### ANALYZING SLEEPING PATTERNS COLLECTED BY FITBIT USING FITBITR

__Setup fitbit API__: 

  Go to teramonagi github page
  on the fitbitr package
   -->     https://github.com/teramonagi/fitbitr
   And follow the instructions to set up the fitbit API
  
  
#### THEN: Get auth
```
  # install.packages("devtools")
  # devtools::install_github("bitowaqr/fitbitr") 
  library(fitbitr)

  FITBIT_KEY    <- "your_key"
  FITBIT_SECRET <- "your_secret"
  token <- oauth_token(key = FITBIT_KEY, secret = FITBIT_SECRET)
```

#### Use the 'sleep_analyzer' function:

__sleep_analyzer()__

* Check out the description: `?sleep_analyze`
* Inspired by mmparker (https://gist.github.com/mmparker/54f1fd05e62671f80dfb4b855231e6ee)  
* See the function's source code here: https://github.com/bitowaqr/fitbitr/blob/master/R/sleep.R  



```
# Investigating how my sleep differs between workdays and weekends:
test = sleep_analyzer(start_date="2017-06-01",
                      end_date="2018-06-14",
                      limit = c("Monday","Tuesday" , "Wednesday", "Thursday","Friday"), 
                      contrast = c("Saturday",  "Sunday"  ),
                      smooth_span = 0.25,
                      bar_spacing = 0.1)

# overview
test$sleep_plot
# sleep contrasts
test$sleep_contrasts_plot
# sleep duration over time
test$sleep_over_time  
# sleep duration frequencies
test$histogram
# boxplot: sleep duration per weekday
test$boxplot
# testing: differences in sleep duration per weekday
summary(test$lm_day)
# testing: differences in sleep duration per month
summary(test$lm_month)
# testing: differences between contrasts
summary(test$lm_contrast)
# raw data set
head(test$raw_data)

```

Have a good sleep! ; )



