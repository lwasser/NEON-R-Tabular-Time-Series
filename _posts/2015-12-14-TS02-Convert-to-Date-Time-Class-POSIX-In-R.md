---
layout: post
title: "Lesson 02: Convert Date & Time Data from Character Class to Date-Time Class (POSIX) in R"
date: 2015-10-23
authors: [Megan A. Jones, Marisa Guarinello, Courtney Soderberg]
contributors: [Leah A. Wasser]
dateCreated: 2015-10-22
lastModified: 2015-12-14
packagesLibraries: [lubridate]
tags: module-1
description: "This lesson explain how to prepare tabular data for further
analysis in `R` through understanding date-time classes and learning how to
convert date-time character class data to a POSIX date-time class."
code1: TS02-Convert-to-Date-Time-Class-POSIX-In-R.R
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink: http://www.neoninc.org
permalink: /R/Time-Series-Convert-Date-Time-Class-POSIX
comments: false
---

{% include _toc.html %}

##About
This lesson explain how to prepare tabular data for further analysis in `R` 
through understanding date-time classes and learning how to convert date-time 
character class data to a POSIX date-time class.

**R Skill Level:** Intermediate - you've got the basics of `R` down and 
understand the general structure of tabular data.

<div id="objectives" markdown="1">

### Goals / Objectives
After completing this activity, you will:

 * Be able to examine data structures and classes within `R`.
 * Understand what POSIXct and POSIXlt data classes are and why we are using
 POSIXct. 
 * Be aware of other date-time classes. 
 * Be able to convert data between different date-time classes. 
 * Be able to designate which format a date-time class will be displayed in. 

###Challenge Code
Throughout the lesson we have Challenges that reinforce learned skills. Possible
solutions to the challenges are not posted on this page, however, the code for
each challenge is in the `R` code that can be downloaded for this lesson (see
footer on this page).

###Things You'll Need To Complete This Lesson
Please be sure you have the most current version of `R` and, preferably,
RStudio to write your code.

####R Libraries to Install
<li><strong>lubridate:</strong> <code> install.packages("lubridate")</code></li>

####Data to Download
<a href="https://ndownloader.figshare.com/files/3579861" class="btn btn-success">
Download Atmospheric Data</a>

The data used in this lesson were collected at Harvard Forest which is
an National Ecological Observatory Network  
<a href="http://www.neoninc.org/science-design/field-sites/harvard-forest" target="_blank"> field site</a>. These data are proxy data for what will be available for 30 years on the [NEON data portal](http://data.neoninc.org/ "NEON data")
for both Harvard Forest and other field sites located across the United States.

####Setting the Working Directory
The code in this lesson assumes that you have set your working directory to the
location of the unzipped file of data downloaded above.  If you would like a
refresher on setting the working directory, please view the [Setting A Working Directory In R]({{site.baseurl}}/R/Set-Working-Directory/ "R Working Directory Lesson") lesson prior to beginning
this lesson.

###Time Series Lesson Series 
This lesson is a part of a series of lessons on tabular time series data in `R`:

* [ Brief Time Series in R - Simple Plots with qplot & as.Date Conversion]({{ site.baseurl}}/R/Brief-Tabular-Time-Series-qplot/)
* [Understanding Time Series Metadata]({{ site.baseurl}}/R/Time-Series-Metadata/)
* [Convert Date & Time Data from Character Class to Date-Time Class (POSIX) in R]({{ site.baseurl}}R/Time-Series-Convert-Date-Time-Class-POSIX/)
* [Refining Time Series Data - NoData Values and Subsetting Data by Date in R]({{ site.baseurl}}/R/Subset-Data-and-No-Data-Values/)
* [Subset and Manipulate Time Series Data with dplyr]({{ site.baseurl}}/R/Time-Series-Subset-dplyr/)
* [Plotting Time Series with ggplot in R]({{ site.baseurl}}/R/Time-Series-Plot-ggplot/)
* [Plot uisng Facets and Plot Time Sereis with NDVI data]({{ site.baseurl}}/R/Time-Series-Plot-Facets-NDVI/)
* [Converting to Julian Day]({{ site.baseurl}}/R/julian-day-conversion/)

###Sources of Additional Information

</div>

#The Data Approach
In [Brief Tabular Time-Series: Plot with qPlot ]({{site.baseurl}}/R/Brief-Tabular-Time-Series-qplot/ "Lesson 00")
we imported a
`.csv` time series dataset and learned how to cleanly plot these data by 
converting the date column to an `R` `Date` data class.  In this lesson we will 
explore how to work with data that contains both a date AND a time stamp.

We will be using functions from both base `R` and the `lubridate` 
package to learn how to work with date-time data classes.

We will use the `hf001-10-15min-m.csv` file that contains micrometeoreology
data including our variables of interest: temperature, precipitation and 
photosynthetically active radiation (PAR), for Harvard Forest, aggregated at
15-minute intervals. If you have not previously loaded this file, we will do so 
now. 


    # Load packages required for entire script
    library(lubridate)  #work with dates
    
    #set working directory to ensure R can find the file we wish to import
    #setwd("working-dir-path-here")
    
    #Load csv file of 15 min meteorological data from Harvard Forest
    #Factors=FALSE so strings, series of letters/ words/ numerals, remain characters
    harMet_15Min <- read.csv(file="AtmosData/HARV/hf001-10-15min-m.csv",
                         stringsAsFactors = FALSE)

##Date and Time Data

Let's revisit the data structure, looking specifically at the `date-time` data.
It is in which data class? 


    #view column data class
    class(harMet_15Min$datetime)

    ## [1] "character"

    #view sample data
    head(harMet_15Min$datetime)

    ## [1] "2005-01-01T00:15" "2005-01-01T00:30" "2005-01-01T00:45"
    ## [4] "2005-01-01T01:00" "2005-01-01T01:15" "2005-01-01T01:30"

Our `datetime` column is in a character data type. We need to convert it to 
date-time data class. What happens when we use the `as.Date` method we learned
in the [Brief Tabular Time Series lesson ]({{site.baseurl}}/R/Brief-Tabular-Time-Series-qplot/ "for as.Date") 
on our `datetime` column?


    #convert column to date class
    har_dateOnly <- as.Date(harMet_15Min$datetime)
    
    #view data
    head(har_dateOnly)

    ## [1] "2005-01-01" "2005-01-01" "2005-01-01" "2005-01-01" "2005-01-01"
    ## [6] "2005-01-01"

We've lost the time stamp associated with the `datetime` data. The `as.Date` 
method worked great for data with just dates, but it doesn't store any time
information. Let's take a minute to explore the various date/time classes in
`R`.

## Exploring Date and Time Classes

###R - Date Class - as.Date
As we just saw, when we use the`as.Date` method to convert date data that is
a string/character class to an `R` `date` class, it will ignore all values after
the date string.

Let's explore date classes in more detail separately from our data set.


    #Convert character data to date (no time) 
    myDate <- as.Date("2015-10-19 10:15")   
    str(myDate)

    ##  Date[1:1], format: "2015-10-19"

    #what happens if the date has text at the end?
    myDate2 <- as.Date("2015-10-19Hello")   
    str(myDate2)

    ##  Date[1:1], format: "2015-10-19"

As we can see above the `as.Date()` function will convert the characters that it
recognizes to be part of a date into a date class and ignore all other
characters in the string. 

###R - Date-Time - The POSIX classes
Given that we do have data and time that are both important for our time series 
we need to use a data class that includes time and date.  Base `R` offers two 
closely related classes for date and time: `POSIXct` and `POSIXlt`. Let's 
explore both of these before choosing which one to use with our data.

The `as.POSIXct` method converts a date-time string into a `POSIXct` class. 

    #Convert character data to date and time.
    timeDate <- as.POSIXct("2015-10-19 10:15")   
    str(timeDate)

    ##  POSIXct[1:1], format: "2015-10-19 10:15:00"

    timeDate

    ## [1] "2015-10-19 10:15:00 MDT"

We see the date and time with a time zone designation - the time zone your
computer is set to, likely your local time zone. 

POSIXct stores date and time as the number of seconds since 1 January 1970
(negative numbers are used for dates before 1970). As each date and time is 
just a single number this class is easy to use in data frames and for a computer
to convert from one format to another. 


    #to see the data in this 'raw' format, i.e., not formatted according to the 
    #class type to show us a date we recognize, use the `unclass()` function.
    unclass(timeDate)

    ## [1] 1445271300
    ## attr(,"tzone")
    ## [1] ""

Here we see the number of seconds from 1 January 1970 to 9 October 2015 and
see that it has a time zone attribute stored with it. 

While easy for data storage and manipulations, humans aren't so good at working
with seconds from 1970 to figure out a date. Plus, we often want to pull out
some portion of the data (e.g., months). The POSIXlt class stores date and time
information in a format that we are used to seeing (e.g., 
second, min, hour, day of month, month, year, numeric day of year, etc). 


    #Convert character data to POSIXlt date and time
    timeDatelt<- as.POSIXlt("2015-10-19 10:15")  
    str(timeDatelt)

    ##  POSIXlt[1:1], format: "2015-10-19 10:15:00"

    timeDatelt

    ## [1] "2015-10-19 10:15:00 MDT"

    unclass(timeDatelt)

    ## $sec
    ## [1] 0
    ## 
    ## $min
    ## [1] 15
    ## 
    ## $hour
    ## [1] 10
    ## 
    ## $mday
    ## [1] 19
    ## 
    ## $mon
    ## [1] 9
    ## 
    ## $year
    ## [1] 115
    ## 
    ## $wday
    ## [1] 1
    ## 
    ## $yday
    ## [1] 291
    ## 
    ## $isdst
    ## [1] 1
    ## 
    ## $zone
    ## [1] "MDT"
    ## 
    ## $gmtoff
    ## [1] NA

When we converted the string to `POSIXlt`, it still appears to be the same as 
`POSIXct`. However, `unclass()` shows us that the data are stored differently. 
The `POSIXlt` class stores the hour, minute, second, day, month, and year as 
separate components.

Yet, just like `POSIXct` being based on seconds since 1 Jan. 1970, `POSIXlt` has
a few quirks.  First, the months use zero based indexing which starts counting
with zero instead of 1, so October is stored as the 9th month (`$mon = 9`) and 
the years are based on years since 1900, so 2015 is stored as 115 (`$year = 115`)
Information like this can always be found in the `R` documentation of a package
or function by using `?` (here, `?POSIXlt`).  

Having dates classified as seperate components makes `POSIXlt` computationally
more difficult to use in data frames, which is why we won't use POSIXlt for this
lesson.  

Instead we can specified which format (e.g., year-month-day hour:minute or 
day/month/year hour:minute) we wanted to see the date and time when using
POSIXct. 

###R- Data-Time - Other Classes
There are many other options in `R` for date-time data classes, including 
packages `readr` and `zoo`. The `chron` package allows the use of time
without a date attached.  

##Convert to a Date-time Class

### Date-Time Formats
We can tell R how our data are formatted (or, in other contexts, how we want to
view date and time) using a `format=` string.  But first we have to know how our
data are formated. 


    #view one date time entry
    harMet_15Min$datetime[1]

    ## [1] "2005-01-01T00:15"

Our data is in Year-Month-Day "T" Hour:Minute:Second, so we can use the 
following designations for the components of the date-time data:

* %Y - year 
* %m - month
* %d - day
* %H:%M - hours:minutes

Bonus: A list of options for the date-time formatting are given in the help for 
the `strptime` function (`help(strptime)`).   {: .notice2}

The "T" inserted into the middle of datetime isn't a standard character for date 
and time, however we can tell `R` where the character is so it can ignore it and
interpret the correct date and time.  


    #convert single instance of date/time in format year-month-day hour:min:sec
    as.POSIXct(harMet_15Min$datetime[1],format="%Y-%m-%dT%H:%M")

    ## [1] "2005-01-01 00:15:00 MST"

    ##The format of date-time MUST match the specified format or the data will not
    # convert; see what happens when you try it a different way or without the "T"
    #specified
    as.POSIXct(harMet_15Min$datetime[1],format="%d-%m-%Y%H:%M")

    ## [1] NA

    as.POSIXct(harMet_15Min$datetime[1],format="%Y-%m-%d%H:%M")

    ## [1] NA

Using the syntax we've learned, we can convert the entire `datetime` column into 
`POSIXct` data class.

##Select a Time Zone
Whenever using time data it is essential to know what time zone the
data were collected in and in what time zone they are stored within `R`.  Data 
may be collected and/or stored in a local time zone but storage in Coordinated 
Universal Time (UTC) or GMT is a common default. 


    #checking time zone of data as R sees it
    tz(harMet_15Min)

    ## [1] "UTC"

`R` currently interprets that the time data are record in UTC. We need to make 
sure it corresponds with what we know is true for the data.  In the [Understanding Time Series Metadata lesson ]({{site.baseurl}}/R/Time-Series-Metadata/ "Time Series Metadata") about the 
associated metadata, we learned that the data were collected in Eastern Standard 
Time.  

We need to designate the correct time zone when converting the data to POSIXct.
There are individual designations for different time zones and time zone 
variants (e.g., is daylight savings time observed). All of the codes can be
found in this <a href="https://en.wikipedia.org/wiki/List_of_tz_database_time_zones" target="_blank"> time zone table</a>. 

The code for the Eastern time zone that is the closest match to the Harvard
Forest is `America/New_York`. 

To try this out let's convert only the first entry to POSIXct specifying the
format and the correct time zone. 


    #assign time zone to just the first entry
    as.POSIXct(harMet_15Min$datetime[1],
                format = "%Y-%m-%dT%H:%M",
                tz = "America/New_York")

    ## [1] "2005-01-01 00:15:00 EST"

We can see that the time zone is now correctly set as EST.  

## Convert to Date-time Data Class
Now, using the syntax that we learned above, we can convert the entire `datetime`
column in the object to an `POSIXct` class.


    #convert to POSIXct date-time class
    harMet_15Min$datetime <- as.POSIXct(harMet_15Min$datetime,
                                    format = "%Y-%m-%dT%H:%M",
                                    tz = "America/New_York")
    
    #view structure and time zone of the newly defined datetime column
    str(harMet_15Min$datetime)

    ##  POSIXct[1:376800], format: "2005-01-01 00:15:00" "2005-01-01 00:30:00" ...

    tz(harMet_15Min$datetime)

    ## [1] "America/New_York"

Now that our `datetime` data is properly identified as a `POSIXct` date-time
data class we can continue on and look at the patterns of precipitation,
temperature, and PAR through time. 