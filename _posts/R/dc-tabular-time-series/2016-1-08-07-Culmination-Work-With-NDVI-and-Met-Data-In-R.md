---
layout: post
title: "Time Series Culmination Activity: Plot using Facets & Plot NDVI with Time Series Data"
date:   2015-10-18
authors: [Megan A. Jones, Leah A. Wasser]
contributors: []
dateCreated: 2015-10-22
lastModified: 2016-03-08
packagesLibraries: [ggplot2, scales, gridExtra, grid, dplyr, reshape2]
categories: [self-paced-tutorial]
mainTag: tabular-time-series
tags: [time-series, phenology, R]
tutorialSeries: [tabular-time-series]
description: "This tutorial is a data integration wrap-up culmination activity 
for the spatio-temporal time series tutorials."
code1: TSCulmination-Work-With-NDVI-and-Met-Data-In-R.R
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink: http://www.neoninc.org
permalink: /R/Work-With-NDVI-and-Met-Data-In-R/
comments: true
---

{% include _toc.html %}

## About
This tutorial is a culmination activity for the series on 
[working with tabular time series data in R ]({{ site.baseurl }}/tutorial-series/tabular-time-series/) 
and as part of a larger spatio-temporal tutorial series and Data Carpentry
workshop on 
[using spatio-temporal phenology-related data in R ]({{ site.baseurl }}/tutorial-series/neon-dc-phenology-series/).
The data used in this culmination activity has been extracted or previously
worked with in the tutorial series
[working with tabular time series data in R ]({{ site.baseurl }}/tutorial-series/tabular-time-series/)
and 
[working with raster time-series data in R ]({{ site.baseurl }}/tutorial-series/raster-time-series/).

**R Skill Level:** Advanced/ Intermediate - you've got the basics of `R` down
 and are comfortable using `ggplot2` and `dplyr`. 

<div id="objectives" markdown="1">

# Goals / Objectives
After completing this activity, you will:

 * have applied `ggplot2` and `dplyr` skills to a new data set.
 * learn how to set min/max axis values in `ggplot()` to align data on multiple plots. 

## Things You’ll Need To Complete This Tutorial
You will need the most current version of R and, preferably, RStudio loaded on
your computer to complete this tutorial.

###Install R Packages
* **ggplot2:** `install.packages("ggplot2")`
* **scales:** `install.packages("scales")`
* **gridExtra:** `install.packages("gridExtra")`
* **grid:** `install.packages("grid")`
* **dplyr:** `install.packages("dplyr")`
* **reshape2:** `install.packages("reshape2")`

[More on Packages in R - Adapted from Software Carpentry.]({{ site.baseurl }}/R/Packages-In-R/)

### Download Data 
{% include/dataSubsets/_data_Met-Time-Series.html %}

****

{% include/_greyBox-wd-rscript.html %}

****

### Recommended Tutorials
This tutorial relies on participants' familiarity with both `dplyr` and 
`ggplot2`. Prior to working through this culmination activity, we recommend the
following tutorials, if you are new to either of the `R` packages.

* [Subset & Manipulate Time Series Data with dplyr tutorial ]({{ site.baseurl }}/R/time-series-subset-dplyr/ "Learn dplyr") 

* [Plotting Time Series with ggplot in R tutorial ]({{ site.baseurl }}/R/time-series-plot-ggplot/ "Learn ggplot")  

</div>

## Plot NDVI & PAR using Daily Data

### NDVI Data
Normalized Difference Vegetation Index (NDVI) is an indicator of how green
vegetation is.  

<iframe width="560" height="315" src="https://www.youtube.com/embed/rxOMhQwApMc" frameborder="0" allowfullscreen></iframe>

NDVI is derived from remote sensing data based on a ratio the
reluctance of visible red spectra and near-infrared spectra.  The NDVI values
vary from -1.0 to 1.0.

The imagery data used to create this NDVI data were collected over the National
Ecological Observatory Network's
<a href="http://www.neonscience.org/science-design/field-sites/harvard-forest" target="_blank" >Harvard Forest</a>
field site.  
The imagery was created by the U.S. Geological Survey (USGS) using a 
<a href="http://eros.usgs.gov/#/Find_Data/Products_and_Data_Available/MSS" target="_blank" >  multispectral scanner</a>
on a
<a href="http://landsat.usgs.gov" target="_blank" > Landsat Satellite </a>.
The data files are Geographic Tagged Image-File Format (GeoTIFF).  
The tutorial 
[Extract NDVI Summary Values from a Raster Time Series]({{ site.baseurl }}/R/Extract-NDVI-From-Rasters-In-R/), 
explains how to create this NDVI file from raster data. 

### Read In the Data
We need to read in two datasets: the 2009-2011 micrometeorological data and the
2011 NDVI data for the Harvard Forest. 


    # Remember it is good coding technique to add additional libraries to the top of
    # your script 
    
    library(lubridate) # for working with dates
    library(ggplot2)  # for creating graphs
    library(scales)   # to access breaks/formatting functions
    library(gridExtra) # for arranging plots
    library(grid)   # for arrangeing plots
    library(dplyr)  # for subsetting by season
    
    # set working directory to ensure R can find the file we wish to import
    # setwd("working-dir-path-here")
    
    # read in the Harvard micro-meteorological data; if you don't already have it
    harMetDaily.09.11 <- read.csv(
      file="NEON-DS-Met-Time-Series/HARV/FisherTower-Met/Met_HARV_Daily_2009_2011.csv",
      stringsAsFactors = FALSE
      )
    
    #check out the data
    str(harMetDaily.09.11)

    ## 'data.frame':	1095 obs. of  47 variables:
    ##  $ X        : int  2882 2883 2884 2885 2886 2887 2888 2889 2890 2891 ...
    ##  $ date     : chr  "2009-01-01" "2009-01-02" "2009-01-03" "2009-01-04" ...
    ##  $ jd       : int  1 2 3 4 5 6 7 8 9 10 ...
    ##  $ airt     : num  -15.1 -9.1 -5.5 -6.4 -2.4 -4.9 -2.6 -3.2 -9.9 -11.1 ...
    ##  $ f.airt   : chr  "" "" "" "" ...
    ##  $ airtmax  : num  -9.2 -3.7 -1.6 0 0.7 0 -0.2 -0.5 -6.1 -8 ...
    ##  $ f.airtmax: chr  "" "" "" "" ...
    ##  $ airtmin  : num  -19.1 -15.8 -9.5 -11.4 -6.4 -10.1 -5.1 -9.9 -12.5 -15.9 ...
    ##  $ f.airtmin: chr  "" "" "" "" ...
    ##  $ rh       : int  58 75 69 59 77 65 97 93 78 77 ...
    ##  $ f.rh     : chr  "" "" "" "" ...
    ##  $ rhmax    : int  76 97 97 78 97 88 100 100 89 92 ...
    ##  $ f.rhmax  : chr  "" "" "" "" ...
    ##  $ rhmin    : int  33 47 41 40 45 38 77 76 63 54 ...
    ##  $ f.rhmin  : chr  "" "" "" "" ...
    ##  $ dewp     : num  -21.9 -12.9 -10.9 -13.3 -6.2 -10.9 -3 -4.2 -13.1 -14.5 ...
    ##  $ f.dewp   : chr  "" "" "" "" ...
    ##  $ dewpmax  : num  -20.4 -6.2 -6.4 -9.1 -1.7 -7.5 -0.5 -0.6 -11.2 -10.5 ...
    ##  $ f.dewpmax: chr  "" "" "" "" ...
    ##  $ dewpmin  : num  -23.5 -21 -14.3 -16.3 -12.1 -13 -7.6 -11.8 -15.2 -18 ...
    ##  $ f.dewpmin: chr  "" "" "" "" ...
    ##  $ prec     : num  0 0 0 0 1 0 26.2 0.8 0 1 ...
    ##  $ f.prec   : chr  "" "" "" "" ...
    ##  $ slrt     : num  8.4 3.7 8.1 8.3 2.9 6.3 0.8 2.8 8.8 5.7 ...
    ##  $ f.slrt   : chr  "" "" "" "" ...
    ##  $ part     : num  16.7 7.3 14.8 16.2 5.4 11.7 1.8 7 18.2 11.4 ...
    ##  $ f.part   : chr  "" "" "" "" ...
    ##  $ netr     : num  -39.4 -16.6 -35.3 -24.7 -19.4 -18.9 5.6 -21.7 -31.1 -16 ...
    ##  $ f.netr   : chr  "" "" "" "" ...
    ##  $ bar      : int  1011 1005 1004 1008 1006 1009 991 987 1005 1015 ...
    ##  $ f.bar    : chr  "" "" "" "" ...
    ##  $ wspd     : num  2.4 1.4 2.7 1.9 2.1 1 1.4 0 1.3 1 ...
    ##  $ f.wspd   : chr  "" "" "" "" ...
    ##  $ wres     : num  2.1 1 2.5 1.6 1.9 0.7 1.3 0 1.1 0.6 ...
    ##  $ f.wres   : chr  "" "" "" "" ...
    ##  $ wdir     : int  294 237 278 292 268 257 86 0 273 321 ...
    ##  $ f.wdir   : chr  "" "" "" "" ...
    ##  $ wdev     : int  29 42 24 31 26 44 24 0 20 50 ...
    ##  $ f.wdev   : chr  "" "" "" "" ...
    ##  $ gspd     : num  13.4 8.1 13.9 8 11.6 5.1 9.1 0 10.1 5 ...
    ##  $ f.gspd   : chr  "" "" "" "" ...
    ##  $ s10t     : num  1 1 1 1 1 1 1.1 1.2 1.4 1.3 ...
    ##  $ f.s10t   : logi  NA NA NA NA NA NA ...
    ##  $ s10tmax  : num  1.1 1 1 1 1.1 1.1 1.1 1.3 1.4 1.4 ...
    ##  $ f.s10tmax: logi  NA NA NA NA NA NA ...
    ##  $ s10tmin  : num  1 1 1 1 1 1 1 1.1 1.3 1.2 ...
    ##  $ f.s10tmin: logi  NA NA NA NA NA NA ...

    head(harMetDaily.09.11)

    ##      X       date jd  airt f.airt airtmax f.airtmax airtmin f.airtmin rh
    ## 1 2882 2009-01-01  1 -15.1           -9.2             -19.1           58
    ## 2 2883 2009-01-02  2  -9.1           -3.7             -15.8           75
    ## 3 2884 2009-01-03  3  -5.5           -1.6              -9.5           69
    ## 4 2885 2009-01-04  4  -6.4            0.0             -11.4           59
    ## 5 2886 2009-01-05  5  -2.4            0.7              -6.4           77
    ## 6 2887 2009-01-06  6  -4.9            0.0             -10.1           65
    ##   f.rh rhmax f.rhmax rhmin f.rhmin  dewp f.dewp dewpmax f.dewpmax dewpmin
    ## 1         76            33         -21.9          -20.4             -23.5
    ## 2         97            47         -12.9           -6.2             -21.0
    ## 3         97            41         -10.9           -6.4             -14.3
    ## 4         78            40         -13.3           -9.1             -16.3
    ## 5         97            45          -6.2           -1.7             -12.1
    ## 6         88            38         -10.9           -7.5             -13.0
    ##   f.dewpmin prec f.prec slrt f.slrt part f.part  netr f.netr  bar f.bar
    ## 1              0         8.4        16.7        -39.4        1011      
    ## 2              0         3.7         7.3        -16.6        1005      
    ## 3              0         8.1        14.8        -35.3        1004      
    ## 4              0         8.3        16.2        -24.7        1008      
    ## 5              1         2.9         5.4        -19.4        1006      
    ## 6              0         6.3        11.7        -18.9        1009      
    ##   wspd f.wspd wres f.wres wdir f.wdir wdev f.wdev gspd f.gspd s10t f.s10t
    ## 1  2.4         2.1         294          29        13.4           1     NA
    ## 2  1.4         1.0         237          42         8.1           1     NA
    ## 3  2.7         2.5         278          24        13.9           1     NA
    ## 4  1.9         1.6         292          31         8.0           1     NA
    ## 5  2.1         1.9         268          26        11.6           1     NA
    ## 6  1.0         0.7         257          44         5.1           1     NA
    ##   s10tmax f.s10tmax s10tmin f.s10tmin
    ## 1     1.1        NA       1        NA
    ## 2     1.0        NA       1        NA
    ## 3     1.0        NA       1        NA
    ## 4     1.0        NA       1        NA
    ## 5     1.1        NA       1        NA
    ## 6     1.1        NA       1        NA

    # read in the NDVI CSV data; if you dont' already have it 
    NDVI.2011 <- read.csv(
      file="NEON-DS-Met-Time-Series/HARV/NDVI/meanNDVI_HARV_2011.csv", 
      stringsAsFactors = FALSE
      )
    
    # check out the data
    str(NDVI.2011)

    ## 'data.frame':	11 obs. of  6 variables:
    ##  $ X        : int  1 2 3 4 5 6 7 8 9 10 ...
    ##  $ meanNDVI : num  0.365 0.243 0.251 0.599 0.879 ...
    ##  $ site     : chr  "HARV" "HARV" "HARV" "HARV" ...
    ##  $ year     : int  2011 2011 2011 2011 2011 2011 2011 2011 2011 2011 ...
    ##  $ julianDay: int  5 37 85 133 181 197 213 229 245 261 ...
    ##  $ Date     : chr  "2011-01-05" "2011-02-06" "2011-03-26" "2011-05-13" ...

    head(NDVI.2011)

    ##   X meanNDVI site year julianDay       Date
    ## 1 1 0.365150 HARV 2011         5 2011-01-05
    ## 2 2 0.242645 HARV 2011        37 2011-02-06
    ## 3 3 0.251390 HARV 2011        85 2011-03-26
    ## 4 4 0.599300 HARV 2011       133 2011-05-13
    ## 5 5 0.878725 HARV 2011       181 2011-06-30
    ## 6 6 0.893250 HARV 2011       197 2011-07-16

In the NDVI dataset, we have the following variables:

* 'X': an integer identifying each row
* meanNDVI: the daily total NDVI for that area. (It is a mean of all pixels in
the original raster).
* site: "HARV" means all NDVI values are from the Harvard Forest
* year: "2011" all values are from 2011
* julianDay: the numeric day of the year
* Date: a date in format "YYYY-MM-DD"; currently in **chr** class

<div id="challenge" markdown="1">
## Challenge:  Class Conversion & Subset by Time
The goal of this challenge is to get our data sets ready so that we can work 
with data from each, within the same plots or analyses.  

1. Ensure that date fields within both data sets are in the Date class. If not,
convert the data to the Date class. 

2. The NDVI data is limited to 2011, however, the meteorological data is from
2009-2011. Subset and retain only the 2011 meteorological data. Name it
`harMet.daily2011`.

HINT: If you are having trouble subsetting the data, refer back to
[Subset & Manipulate Time Series Data with dplyr tutorial]({{ site.baseurl }}/R/time-series-subset-dplyr/ "Learn dplyr")
</div>



Now that we have our data sets with Date class dates and limited to 2011, we can
begin working with both. 

## Plot NDVI Data from a .csv
This NDVI data was derived from a raster, however, now it is integers in a
`data.frame` therefore we can plot it like any of our other values using
`ggplot()`. Here we plot `meanNDVI` by `Date`.


    # plot NDVI
    ggplot(NDVI.2011, aes(Date, meanNDVI))+
      geom_point(colour = "forestgreen", size = 4) +
      ggtitle("Daily NDVI at Harvard Forest, 2011")+
      theme(legend.position = "none",
            plot.title = element_text(lineheight=.8, face="bold",size = 20),
            text = element_text(size=20))

![ ]({{ site.baseurl }}/images/rfigs/dc-tabular-time-series/07-Culmination-Work-With-NDVI-and-Met-Data-In-R/plot-NDVI-1.png)

## Two y-axes or Side-by-Side Plots?
When we have different types of data like NDVI (scale: 0-1 index units),
Photosynthetically Active Radiation (PAR, scale: 0-65.8 mole per meter squared),
or temperature (scale: -20 to 30 C) that we want to plot over time, we cannot
simply plot them on the same plot as they have different y-axes.

One option, would be to plot both data types in the same plot space but each
having it's own axis (one on left of plot and one on right of plot).  However, 
there is a line of graphical representation thought that this is not a good
practice.  The creator of `ggplot2` ascribes to this dislike of different y-axes
and so neither `qplot` nor `ggplot` have this functionality. 

Instead, plots of different types of data can be plotted next to each other to 
allow for comparison.  Depending on how the plots are being viewed, they can
have a vertical or horizontal arrangement. 

<div id="challenge" markdown="1">
## Challenge: Plot Air Temperature and NDVI
Then we will plot the two plots in the same viewer so we can more easily compare
them.  

Hint: If you are having a hard time arranging the plots in a single grid, refer
back to 
[Plotting Time Series with ggplot in R tutorial]({{ site.baseurl }}/R/time-series-plot-ggplot/ "Learn ggplot")
</div>


    # plot NDVI again
    plot.NDVI.2011 <- ggplot(NDVI.2011, aes(Date, meanNDVI))+
      geom_point(colour = "forestgreen", size = 4) +
      ggtitle("Daily NDVI at Harvard Forest, 2011")+
      theme(legend.position = "none",
            plot.title = element_text(lineheight=.8, face="bold",size = 20),
            text = element_text(size=20))
    
    # create plot of julian day vs. PAR
    plot.par.2011 <- ggplot(harMet.daily2011, aes(date, part))+
      geom_point(na.rm=TRUE) +
      ggtitle("Daily PAR at Harvard Forest, 2011")+
      theme(legend.position = "none",
            plot.title = element_text(lineheight=.8, face="bold",size = 20),
            text = element_text(size=20))
    
    # display the plots together
    grid.arrange(plot.par.2011, plot.NDVI.2011) 

![ ]({{ site.baseurl }}/images/rfigs/dc-tabular-time-series/07-Culmination-Work-With-NDVI-and-Met-Data-In-R/plot-PAR-NDVI-1.png)

This is nice but a bit confusing as the date on our x-axis doesn't exactly line
up. To fix this we can assign the same min and max to both x-axes so that they
align.  We can assign the date format for the x-axis and clearly label both
axes. 


    # plot PAR
    plot2.par.2011 <- plot.par.2011 +
                   scale_x_date(labels = date_format("%b %d"),
                   date_breaks = "3 months",
                   date_minor_breaks= "1 week",
                   limits=c(min=min(NDVI.2011$Date),max=max(NDVI.2011$Date))) +
                   ylab("Total PAR") + xlab ("")
    
    # plot NDVI
    plot2.NDVI.2011 <- plot.NDVI.2011 +
                   scale_x_date(labels = date_format("%b %d"),
                   date_breaks = "3 months", 
                   date_minor_breaks= "1 week",
                   limits=c(min=min(NDVI.2011$Date),max=max(NDVI.2011$Date)))+
                   ylab("Total NDVI") + xlab ("Date")
    
    # Output with both plots
    grid.arrange(plot2.par.2011, plot2.NDVI.2011) 

![ ]({{ site.baseurl }}/images/rfigs/dc-tabular-time-series/07-Culmination-Work-With-NDVI-and-Met-Data-In-R/plot-same-xaxis-1.png)

<div id="challenge" markdown="1">
## Challenge: Plot Air Temperature and NDVI
Create a plot, complementary to those above, showing air temperature (`airt`)
throughout 2011. Choose colors and symbols that show the data well. 

Finally, plot PAR, air temperature, and NDVI in a single pane to ease
comparisons.  
</div>

![ ]({{ site.baseurl }}/images/rfigs/dc-tabular-time-series/07-Culmination-Work-With-NDVI-and-Met-Data-In-R/challengeplot-same-xaxis-1.png)
