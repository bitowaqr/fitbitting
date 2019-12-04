<head>
<style>
.box {
    border-radius: 5px;
    background: lightgray;
    border: 1px solid black;
    float: center;
    font-size: 11px;
    height: auto;
    margin: 20px;
    padding: 5px;
    width: 300px;
}
.content {
    padding: 10px;
}


</style>
</head>
<br>

#### ANALYSING YOUR FITBIT SLEEPING PATTERNS

<h5>
  Using the <a href="https://github.com/teramonagi/fitbitr">FitbitR
package</a>
</h5>
    Paul Schneider  
   
<a href="mailto:p.schneider@sheffield.ac.uk" class="email">p.schneider@sheffield.ac.uk</a>

------------------------------------------------------------------------

<br>
<h5>
This is a brief R-tutorial how to access your fitbit data and analyse
your sleeping patterns. However, before you can retrieve data from
fitbit, you have to set up a new ‘application’. <br>

> Go to
> <a href="https://dev.fitbit.com/apps/new" class="uri">https://dev.fitbit.com/apps/new</a>
>
> Fill in the form (you can simply copy-paste the fields below)
>
> Click `Register`

<div class='box'>
<div class="content">
<p>
<form>
<b>Application Name\*</b>:<br>
<input size="40" readonly readonly type="text"  value="My_application"><br>

<b>Description</b>:<br>
<input size="40" readonly type="text"  value="Application to analyse sleeping patterns"><br>

<b>Application Website</b>:<br>
<input size="40" readonly type="text"  value="https://github.com/bitowaqr/fitbitting"><br>

<b>Organisation</b>:<br>
<input size="40" readonly type="text"  value="Uni Sheffield"><br>

<b>Organisation Website</b>:<br>
<input size="40" readonly type="text"  value="https://github.com/bitowaqr/fitbitting"><br>

<b>Terms Of Service Url\*</b>:<br>
<input size="40" readonly type="text"  value="https://github.com/bitowaqr/fitbitting"><br>

<b>Privacy Policy Url \*</b>:<br>
<input size="40" readonly type="text"  value="https://github.com/bitowaqr/fitbitting"><br>

<input size="40" readonly tYPE="Radio" Name="Sever" Value="Server" Checked>
Server
<input size="40" readonly tYPE="Radio" Name="Sever" Value="Server">
Client
<input size="40" readonly tYPE="Radio" Name="Sever" Value="Server">
Personal <br>

<b>Callback URL </b>:<br>
<input size="40" readonly type="text"  value="http://localhost:1410/"><br>

</form>
</div>
</div>
<br><br>

<h5>
After you have registered your new application, you should find your
OAuth ID and a Client secret, which we will need in a moment (see
below).

<img src="./sample_oauth.png" alt="Sample Oauth" style="border-radius: 5px;margin:20px;border: 1px solid black;">

<br>

<h5>
Now, Use the code below to install a slightly extended version of the
FitbitR package (the original can be found
<a href="https://github.com/teramonagi/fitbitr">here</a>).
</h5>
``` r
install.packages("devtools") # install devtools if needed
devtools::install_github("bitowaqr/fitbitr")
library(fitbitr)
```

<br>

<h5>
Put in your OAuth ID and Client secret to connect with your fitbit data.
</h5>
``` r
  FITBIT_KEY    <- "YOUR_OAUTH_ID"  
  FITBIT_SECRET <- "YOUR_CLIENT_SECRET"
  token <- oauth_token(key = FITBIT_KEY, secret = FITBIT_SECRET)
```

    ## Waiting for authentication in browser...

    ## Press Esc/Ctrl + C to abort

    ## Authentication complete.

<br>

<h5>
You have to log in into your fitbit account, and then it will ask you to
specify what data you want to access. Select whatever you are interested
in, but to analyse sleeping patterns, you just need to select `sleep`,
of course. Then click `Allow`.
</h5>
<img src="./sleep_check.png" alt="Sample Oauth" style="border-radius: 5px;margin:20px;border: 1px solid black;">

<br>

<h5>
Finally, we can extract the sleep data and run some analysis.
</h5>
<h5>
We use an extended version of fitbitr’s `sleep_analyzer(...)` function
(See the helpfile for further info: `?sleep_analyze`). Thank you to A.
for letting me use their data for demo purposes.
</h5>
``` r
# retrieve your data from fitbit
  my_sleep = sleep_analyzer(
    start_date    =  "2017-06-01", # When did you start wearing fitbit?
    end_date      =   "2019-12-01",   # Select yesterday
    # We can set limit and contrast to study the different sleeping
    # patterns on working days and weekends
    limit         =    c("Monday","Tuesday" , "Wednesday", "Thursday","Friday"), 
    contrast      =    c("Saturday",  "Sunday"  ),
    # input the times you intend to go to sleep and wake up
    # to draw 2 reference lines in the plot
    planned_start = 23, planned_end = 7, 
  )

# Have a look at the relevant bits of the extracted data
head(my_sleep$raw_data)[,c(1,4,3)]
```

``` r
# 1. Overview
  # Shows the start (yellow) and the end (red) of sleep for each day
  # the dark blue lines are smooth trend lines
  # the black lines indicate your planned sleeping times
  my_sleep$sleep_plot
```

![](readme_files/figure-markdown_github/unnamed-chunk-4-1.png)

``` r
# 2. Weekdays versus weekends
  # Showing start and end times separately
  my_sleep$sleep_contrasts_plot
```

![](readme_files/figure-markdown_github/unnamed-chunk-4-2.png)

``` r
# 3. sleep duration over time
  # shows the sleep duration for each day over the 
  # entire observation period. A smooth trend line
  # is shown in blue
  my_sleep$sleep_over_time  
```

![](readme_files/figure-markdown_github/unnamed-chunk-5-1.png)

``` r
# 4. sleep duration frequencies
  # A histogram showing the frequency of your sleep durations
  # the vertical line indicates the median sleep duration
  my_sleep$histogram
```

![](readme_files/figure-markdown_github/unnamed-chunk-5-2.png)

``` r
# 5. Sleep by day of the week
  # boxplots show the hours of sleep per day of the week
  my_sleep$boxplot
```

![](readme_files/figure-markdown_github/unnamed-chunk-5-3.png)

``` r
# 6. sleep by month
  # shows the variation of sleep durations by month
  # (useful if you have >1 year of data)
  # most people sleep longer during winter months
  # and shorter in summer 
  my_sleep$sleep_by_month_plot
```

![](readme_files/figure-markdown_github/unnamed-chunk-5-4.png)

<br>

<h5>
That’s it. More analyses may be coming later.
</h5>

------------------------------------------------------------------------

    Paul Schneider - CC-BY
