---
title: "Homework 9"
author: "Anastasia Karp"
date: "2024-02-21"
output:
  html_document: 
    theme: spacelab
    keep_md: true
---



## Instructions
Answer the following questions and complete the exercises in RMarkdown. Please embed all of your code and push your final work to your repository. Your final lab report should be organized, clean, and run free from errors. Remember, you must remove the `#` for the included code chunks to run. Be sure to add your name to the author header above.  

Make sure to use the formatting conventions of RMarkdown to make your report neat and clean!  

## Load the libraries

```r
library(tidyverse)
library(janitor)
library(here)
library(naniar)
```

For this homework, we will take a departure from biological data and use data about California colleges. These data are a subset of the national college scorecard (https://collegescorecard.ed.gov/data/). Load the `ca_college_data.csv` as a new object called `colleges`.

```r
colleges <- read_csv("data/ca_college_data.csv") %>% clean_names()
```

```
## Rows: 341 Columns: 10
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (4): INSTNM, CITY, STABBR, ZIP
## dbl (6): ADM_RATE, SAT_AVG, PCIP26, COSTT4_A, C150_4_POOLED, PFTFTUG1_EF
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

The variables are a bit hard to decipher, here is a key:  

INSTNM: Institution name  
CITY: California city  
STABBR: Location state  
ZIP: Zip code  
ADM_RATE: Admission rate  
SAT_AVG: SAT average score  
PCIP26: Percentage of degrees awarded in Biological And Biomedical Sciences  
COSTT4_A: Annual cost of attendance  
C150_4_POOLED: 4-year completion rate  
PFTFTUG1_EF: Percentage of undergraduate students who are first-time, full-time degree/certificate-seeking undergraduate students  

1. Use your preferred function(s) to have a look at the data and get an idea of its structure. Make sure you summarize NA's and determine whether or not the data are tidy. You may also consider dealing with any naming issues.

```r
glimpse(colleges)
```

```
## Rows: 341
## Columns: 10
## $ instnm        <chr> "Grossmont College", "College of the Sequoias", "College…
## $ city          <chr> "El Cajon", "Visalia", "San Mateo", "Ventura", "Oxnard",…
## $ stabbr        <chr> "CA", "CA", "CA", "CA", "CA", "CA", "CA", "CA", "CA", "C…
## $ zip           <chr> "92020-1799", "93277-2214", "94402-3784", "93003-3872", …
## $ adm_rate      <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …
## $ sat_avg       <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …
## $ pcip26        <dbl> 0.0016, 0.0066, 0.0038, 0.0035, 0.0085, 0.0151, 0.0000, …
## $ costt4_a      <dbl> 7956, 8109, 8278, 8407, 8516, 8577, 8580, 9181, 9281, 93…
## $ c150_4_pooled <dbl> NA, NA, NA, NA, NA, NA, 0.2334, NA, NA, NA, NA, 0.1704, …
## $ pftftug1_ef   <dbl> 0.3546, 0.5413, 0.3567, 0.3824, 0.2753, 0.4286, 0.2307, …
```
NA summary

```r
naniar::miss_var_summary(colleges)
```

```
## # A tibble: 10 × 3
##    variable      n_miss pct_miss
##    <chr>          <int>    <dbl>
##  1 sat_avg          276     80.9
##  2 adm_rate         240     70.4
##  3 c150_4_pooled    221     64.8
##  4 costt4_a         124     36.4
##  5 pftftug1_ef       53     15.5
##  6 pcip26            35     10.3
##  7 instnm             0      0  
##  8 city               0      0  
##  9 stabbr             0      0  
## 10 zip                0      0
```
the data appears to be tidy.


2. Which cities in California have the highest number of colleges?

```r
colleges %>% 
  group_by(city) %>% 
  count() %>% 
  arrange(desc(n))
```

```
## # A tibble: 161 × 2
## # Groups:   city [161]
##    city              n
##    <chr>         <int>
##  1 Los Angeles      24
##  2 San Diego        18
##  3 San Francisco    15
##  4 Sacramento       10
##  5 Berkeley          9
##  6 Oakland           9
##  7 Claremont         7
##  8 Pasadena          6
##  9 Fresno            5
## 10 Irvine            5
## # ℹ 151 more rows
```



3. Based on your answer to #2, make a plot that shows the number of colleges in the top 10 cities.

```r
colleges %>% 
  group_by(city) %>% 
  summarise(n_colleges=n()) %>% 
  top_n(10, n_colleges) %>% 
  arrange(desc(n_colleges)) %>% 
  ggplot(aes(x=city, y=n_colleges))+
  geom_col()
```

![](hw9_files/figure-html/unnamed-chunk-6-1.png)<!-- -->


4. The column `COSTT4_A` is the annual cost of each institution. Which city has the highest average cost? Where is it located?

```r
colleges %>% 
  group_by(city) %>% 
  summarise(avg_annual_cost=mean(costt4_a, na.rm = T)) %>% 
  arrange(desc(avg_annual_cost))
```

```
## # A tibble: 161 × 2
##    city                avg_annual_cost
##    <chr>                         <dbl>
##  1 Claremont                     66498
##  2 Malibu                        66152
##  3 Valencia                      64686
##  4 Orange                        64501
##  5 Redlands                      61542
##  6 Moraga                        61095
##  7 Atherton                      56035
##  8 Thousand Oaks                 54373
##  9 Rancho Palos Verdes           50758
## 10 La Verne                      50603
## # ℹ 151 more rows
```
Claremont appears to have the highest average cost.


5. Based on your answer to #4, make a plot that compares the cost of the individual colleges in the most expensive city. Bonus! Add UC Davis here to see how it compares :>).

```r
colleges %>% 
  filter(city=="Claremont"| instnm=="University of California-Davis") %>% #need to use or; '|' because arent any UCD in claremont, so not '&'
  ggplot(aes(x=instnm, y=costt4_a))+
  geom_col()+
  coord_flip()
```

```
## Warning: Removed 2 rows containing missing values (`position_stack()`).
```

![](hw9_files/figure-html/unnamed-chunk-8-1.png)<!-- -->


6. The column `ADM_RATE` is the admissions rate by college and `C150_4_POOLED` is the four-year completion rate. Use a scatterplot to show the relationship between these two variables. What do you think this means?

```r
colleges %>% 
  ggplot(aes(y=adm_rate, x=c150_4_pooled))+
  geom_point()+
   geom_smooth(method = lm)
```

```
## `geom_smooth()` using formula = 'y ~ x'
```

```
## Warning: Removed 251 rows containing non-finite values (`stat_smooth()`).
```

```
## Warning: Removed 251 rows containing missing values (`geom_point()`).
```

![](hw9_files/figure-html/unnamed-chunk-9-1.png)<!-- -->
The graph shows that as the four-year completion rate increases, the admission rate decreases.


7. Is there a relationship between cost and four-year completion rate? (You don't need to do the stats, just produce a plot). What do you think this means?

```r
colleges %>% 
  ggplot(aes(x=costt4_a, y=c150_4_pooled))+
  geom_point()+
   geom_smooth(method = lm)
```

```
## `geom_smooth()` using formula = 'y ~ x'
```

```
## Warning: Removed 225 rows containing non-finite values (`stat_smooth()`).
```

```
## Warning: Removed 225 rows containing missing values (`geom_point()`).
```

![](hw9_files/figure-html/unnamed-chunk-10-1.png)<!-- -->
The graph shows that as cost increases, so does the four-year completion rate. This could mean that as students pay more for their tuition, they are more inclined to complete their degreee; get their moneys-worth.


8. The column titled `INSTNM` is the institution name. We are only interested in the University of California colleges. Make a new data frame that is restricted to UC institutions. You can remove `Hastings College of Law` and `UC San Francisco` as we are only interested in undergraduate institutions.

```r
univ_calif_final <- colleges %>% 
  filter_all(any_vars(str_detect(., pattern="University of California") ))
```


Remove `Hastings College of Law` and `UC San Francisco` and store the final data frame as a new object `univ_calif_final`.

```r
univ_calif_final %>% 
  filter(city!="San Francisco")
```

```
## # A tibble: 8 × 10
##   instnm       city  stabbr zip   adm_rate sat_avg pcip26 costt4_a c150_4_pooled
##   <chr>        <chr> <chr>  <chr>    <dbl>   <dbl>  <dbl>    <dbl>         <dbl>
## 1 University … La J… CA     92093    0.357    1324  0.216    31043         0.872
## 2 University … Irvi… CA     92697    0.406    1206  0.107    31198         0.876
## 3 University … Rive… CA     92521    0.663    1078  0.149    31494         0.73 
## 4 University … Los … CA     9009…    0.180    1334  0.155    33078         0.911
## 5 University … Davis CA     9561…    0.423    1218  0.198    33904         0.850
## 6 University … Sant… CA     9506…    0.578    1201  0.193    34608         0.776
## 7 University … Berk… CA     94720    0.169    1422  0.105    34924         0.916
## 8 University … Sant… CA     93106    0.358    1281  0.108    34998         0.816
## # ℹ 1 more variable: pftftug1_ef <dbl>
```


Use `separate()` to separate institution name into two new columns "UNIV" and "CAMPUS".

```r
univ_calif_final %>% 
  separate(instnm, into = c("univ", "campus"), sep = "-")
```

```
## # A tibble: 10 × 11
##    univ               campus city  stabbr zip   adm_rate sat_avg pcip26 costt4_a
##    <chr>              <chr>  <chr> <chr>  <chr>    <dbl>   <dbl>  <dbl>    <dbl>
##  1 University of Cal… San D… La J… CA     92093    0.357    1324  0.216    31043
##  2 University of Cal… Irvine Irvi… CA     92697    0.406    1206  0.107    31198
##  3 University of Cal… River… Rive… CA     92521    0.663    1078  0.149    31494
##  4 University of Cal… Los A… Los … CA     9009…    0.180    1334  0.155    33078
##  5 University of Cal… Davis  Davis CA     9561…    0.423    1218  0.198    33904
##  6 University of Cal… Santa… Sant… CA     9506…    0.578    1201  0.193    34608
##  7 University of Cal… Berke… Berk… CA     94720    0.169    1422  0.105    34924
##  8 University of Cal… Santa… Sant… CA     93106    0.358    1281  0.108    34998
##  9 University of Cal… Hasti… San … CA     9410…   NA          NA NA           NA
## 10 University of Cal… San F… San … CA     9414…   NA          NA NA           NA
## # ℹ 2 more variables: c150_4_pooled <dbl>, pftftug1_ef <dbl>
```


9. The column `ADM_RATE` is the admissions rate by campus. Which UC has the lowest and highest admissions rates? Produce a numerical summary and an appropriate plot.

```r
univ_calif_final %>% 
  separate(instnm, into = c("univ", "campus"), sep = "-")%>% 
  filter(campus!=c("Hastings College of Law", "San Francisco")) %>% 
  select(campus, adm_rate) %>% 
  arrange(adm_rate) %>% 
  ggplot(aes(x=campus, y=adm_rate))+
  geom_col()
```

![](hw9_files/figure-html/unnamed-chunk-14-1.png)<!-- -->


10. If you wanted to get a degree in biological or biomedical sciences, which campus confers the majority of these degrees? Produce a numerical summary and an appropriate plot.

```r
univ_calif_final %>% 
  separate(instnm, into = c("univ", "campus"), sep = "-")%>% 
  filter(campus!=c("Hastings College of Law", "San Francisco")) %>% 
  select(campus, pcip26) %>% 
  arrange(pcip26) %>% 
  ggplot(aes(x=campus, y=pcip26))+
  geom_col()
```

![](hw9_files/figure-html/unnamed-chunk-15-1.png)<!-- -->


## Knit Your Output and Post to [GitHub](https://github.com/FRS417-DataScienceBiologists)
