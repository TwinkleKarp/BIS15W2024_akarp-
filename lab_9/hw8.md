---
title: "Homework 8"
author: "Anastasia Karp"
date: "2024-02-29"
output:
  html_document: 
    theme: spacelab
    keep_md: yes
---



## Instructions
Answer the following questions and complete the exercises in RMarkdown. Please embed all of your code and push your final work to your repository. Your final lab report should be organized, clean, and run free from errors. Remember, you must remove the `#` for the included code chunks to run. Be sure to add your name to the author header above.  

Make sure to use the formatting conventions of RMarkdown to make your report neat and clean!  

## Load the libraries

```r
library(tidyverse)
library(janitor)
```

## Install `here`
The package `here` is a nice option for keeping directories clear when loading files. I will demonstrate below and let you decide if it is something you want to use.  

```r
#install.packages("here")
```

## Data
For this homework, we will use a data set compiled by the Office of Environment and Heritage in New South Whales, Australia. It contains the enterococci counts in water samples obtained from Sydney beaches as part of the Beachwatch Water Quality Program. Enterococci are bacteria common in the intestines of mammals; they are rarely present in clean water. So, enterococci values are a measurement of pollution. `cfu` stands for colony forming units and measures the number of viable bacteria in a sample [cfu](https://en.wikipedia.org/wiki/Colony-forming_unit).   

This homework loosely follows the tutorial of [R Ladies Sydney](https://rladiessydney.org/). If you get stuck, check it out!  

1. Start by loading the data `sydneybeaches`. Do some exploratory analysis to get an idea of the data structure.

```r
sydneybeaches <- read_csv("data/sydneybeaches.csv") %>% clean_names()
```

```
## Rows: 3690 Columns: 8
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (4): Region, Council, Site, Date
## dbl (4): BeachId, Longitude, Latitude, Enterococci (cfu/100ml)
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

If you want to try `here`, first notice the output when you load the `here` library. It gives you information on the current working directory. You can then use it to easily and intuitively load files.

```r
#library(here)
```

The quotes show the folder structure from the root directory.

```r
#sydneybeaches <-read_csv(here("homework", "data", "sydneybeaches.csv")) %>% clean_names()
```

2. Are these data "tidy" per the definitions of the tidyverse? How do you know? Are they in wide or long format?

```r
sydneybeaches
```

```
## # A tibble: 3,690 × 8
##    beach_id region  council site  longitude latitude date  enterococci_cfu_100ml
##       <dbl> <chr>   <chr>   <chr>     <dbl>    <dbl> <chr>                 <dbl>
##  1       25 Sydney… Randwi… Clov…      151.    -33.9 02/0…                    19
##  2       25 Sydney… Randwi… Clov…      151.    -33.9 06/0…                     3
##  3       25 Sydney… Randwi… Clov…      151.    -33.9 12/0…                     2
##  4       25 Sydney… Randwi… Clov…      151.    -33.9 18/0…                    13
##  5       25 Sydney… Randwi… Clov…      151.    -33.9 30/0…                     8
##  6       25 Sydney… Randwi… Clov…      151.    -33.9 05/0…                     7
##  7       25 Sydney… Randwi… Clov…      151.    -33.9 11/0…                    11
##  8       25 Sydney… Randwi… Clov…      151.    -33.9 23/0…                    97
##  9       25 Sydney… Randwi… Clov…      151.    -33.9 07/0…                     3
## 10       25 Sydney… Randwi… Clov…      151.    -33.9 25/0…                     0
## # ℹ 3,680 more rows
```
The data are untidy because many of the variables, such as in site, council, and region, are not in their own columns. So, it is in long format.

3. We are only interested in the variables site, date, and enterococci_cfu_100ml. Make a new object focused on these variables only. Name the object `sydneybeaches_long`

```r
sydneybeaches_long <- 
  sydneybeaches %>% 
  select( site, date, enterococci_cfu_100ml)
```


4. Pivot the data such that the dates are column names and each beach only appears once (wide format). Name the object `sydneybeaches_wide`

```r
sydneybeaches_wide <- 
  sydneybeaches_long %>% 
  pivot_wider(names_from = date,
              values_from = enterococci_cfu_100ml)  
```


5. Pivot the data back so that the dates are data and not column names.

```r
sydneybeaches_wide %>% 
  pivot_longer(-site,
               names_to = "date",
               values_to = "enterococci_cfu_100ml")
```

```
## # A tibble: 3,784 × 3
##    site           date       enterococci_cfu_100ml
##    <chr>          <chr>                      <dbl>
##  1 Clovelly Beach 02/01/2013                    19
##  2 Clovelly Beach 06/01/2013                     3
##  3 Clovelly Beach 12/01/2013                     2
##  4 Clovelly Beach 18/01/2013                    13
##  5 Clovelly Beach 30/01/2013                     8
##  6 Clovelly Beach 05/02/2013                     7
##  7 Clovelly Beach 11/02/2013                    11
##  8 Clovelly Beach 23/02/2013                    97
##  9 Clovelly Beach 07/03/2013                     3
## 10 Clovelly Beach 25/03/2013                     0
## # ℹ 3,774 more rows
```


6. We haven't dealt much with dates yet, but separate the date into columns day, month, and year. Do this on the `sydneybeaches_long` data.

```r
sydneybeaches_long_date <- sydneybeaches_long %>% 
  separate(date, into= c("day", "month", "year"), sep = "/")
```


7. What is the average `enterococci_cfu_100ml` by year for each beach. Think about which data you will use- long or wide.

```r
sydneybeaches_long_date_year <- sydneybeaches_long_date %>% 
  group_by(year) %>% 
  summarise(mean_enterococci=mean(enterococci_cfu_100ml, na.rm = T))
```


8. Make the output from question 7 easier to read by pivoting it to wide format.

```r
sydneybeaches_long_date_year %>% 
  pivot_wider(names_from = "year", #the observations under key will become new columns
              values_from = "mean_enterococci") #the values under value will be moved to the new columns)
```

```
## # A tibble: 1 × 6
##   `2013` `2014` `2015` `2016` `2017` `2018`
##    <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
## 1   50.6   26.3   31.2   42.2   20.7   33.1
```


9. What was the most polluted beach in 2013?

```r
sydneybeaches_long_date %>%  
  filter(year=="2013") %>% 
  group_by(site) %>% 
  summarise(highest_pollution=max(enterococci_cfu_100ml)) %>% 
  arrange(desc(highest_pollution))
```

```
## # A tibble: 11 × 2
##    site                    highest_pollution
##    <chr>                               <dbl>
##  1 Little Bay Beach                     4900
##  2 Malabar Beach                        2500
##  3 Maroubra Beach                       2100
##  4 South Maroubra Beach                 1200
##  5 Bronte Beach                          780
##  6 Tamarama Beach                        690
##  7 Gordons Bay (East)                    660
##  8 Coogee Beach                          630
##  9 South Maroubra Rockpool               630
## 10 Bondi Beach                           600
## 11 Clovelly Beach                         97
```
Little Bay Beach was the most polluted beach in 2013.

10. Please complete the class project survey at: [BIS 15L Group Project](https://forms.gle/H2j69Z3ZtbLH3efW6)

## Push your final code to GitHub!
Please be sure that you check the `keep md` file in the knit preferences.   
