---
title: "Measuring and Improving Performance"
author: "Carmen Garro and Luke Smith"
date: "2022-16-11"
output: 
  html_document:
    keep_md: true #output in github
    toc: TRUE #table of contents
    df_print: paged 
    number_sections: FALSE #number headings
    highlight: tango #theme for code syntax
    theme: lumen #theme of markdown
    toc_depth: 3 #depth of headers, sections and subseections to be included
    toc_float: true #put table of contents in left side
    css: custom.css 
    self_contained: false
---

# Introduction

This material is mainly based on the chapters 23 and 24 of Hadley Wickham's book [Advanced R](http://adv-r.had.co.nz/Performance.html) 


## What is performance?

The performance of a code is how quickly a computer can perform a task given certain code. The performance of a code depends on how fast and how many resources are needed for a computational task to be done. It is a balance between deciding the point of efficiency for the code, and how many hours have to be dedicated from the person for this efficiency to be reached.

## Is R an efficient language?

![](C:/Users/carme/Dropbox/MIO/DOCS/Hertie/Semestre/Semestre I/Intro to Data Science/Workshop/performance-chart-langs.png)

R is a language designed to make analytics easier for people, it was not designed with efficiency in mind. This chart is composed by the time it takes for different coding languages to multiply two matrices of dimensions 1000x1000. R is evidently the slowest in the list.

## Why is improving performance important?

There are various reasons for improving performance, specially nowadays with so many tools in hand and the data science and computer science fields broadening. 

* The less efficient the code, the more time it takes processing data i.e., reduce bottlenecks in your code
* Less memory usage
* CO2 emissions from energy usage with high intensity computing

## Environmental considerations

When considering the importance of measuring and improving the performance of your code, it is important to be aware of the environmental aspects associated with intense computing. In a paper titled, "Aligning artificial intelligence with climate change mitigation", from Lynn Kaack, an Assistant Professor of Computer Science and Public Policy at the Hertie School, this is discussed in detail. 

The vast majority of models that have been employed so far in our program are relatively small and their environmental impact incredibly minimal. However, as we seek to become data scientists working at potentially large organizations, it becomes more likely that we will engage with increasingly complex models that utilize a large amount of computing power, and therefore, a large amount of electricity. Professor Kaack highlights neural network models, and the training stage of model development in particular, as a very energy intensive process given the high number of potential model configurations and trial and error continuing to be the most effective way to configure the model.Specific company examples noted in the paper include:

- Google's machine translation system may process more than 100 billion words per day.
- Facebook's datacenters are re-trained anywhere from hourly to multi-monthly.

![](C:/Users/carme/Dropbox/MIO/DOCS/Hertie/Semestre/Semestre I/Intro to Data Science/Workshop/pic for IDS.png)

While it is still not standard practice to consider the environmental impact of machine learning, it will becoming increasingly relevant as AI and machine learning continue to increase in prevalence. While measuring and improving performance will not singlehandedly address environmental concerns, it is a small yet important step in ensuring that your code is as efficient as possible. Not just to reduce the time it takes to run, but to minimize the amount of energy utilized to run it.

# Measuring performance 

Find what could make your code faster first, these are called **bottlenecks**. Don’t rely on intuition, rely on your code **profile**, measure the run-time of each line of code using realistic inputs.

Once these bottleneck are found, you can experiment with alternatives to find faster code.

Steps:

* Dig into what is making the code slower with a profile, with the **profvis** package
* Use microbenchmark from the **banch** package to explore alternative implementations and which one is faster


```r
library(profvis)
library(bench)
```

## Profiling (from profvis package)

A profiler is the primary tool used across programming languages. R uses a sampling or statistical profiler: it stops the execution of code every few milliseconds and records the call stack (i.e. which function is currently executing, and the function that called the function, and so on).

## Benchmark (from bench package)

This library is a tool to accurately analyze and benchmark execution times for R expressions.

[https://github.com/r-lib/bench](https://github.com/r-lib/bench)

Features:

* Always uses the highest precision APIs available for each operating systems (often nanoseconds)
* Tracks memory allocations for each expression
* Tracks the number and type of R garbage collections per expression iteration
* Verifies equality of expression results by default, to avoid accidentally benchmarking inequivalent code
* Uses adaptive stopping by default, running each expression for a set amount of time rather than for a specific number of iterations
* Expressions are sun in batches and summary statistics are calculated after filtering out iterations with garbage collections. This allows you to isolate the performance and effects of garbage collection on running time
* The time and memory usage are returned as custom objects which have human readable formatting for display and comparisons.
* There is also support for plotting with ggplot2, including custom scales.

## Visualizing profiles

Even if your function takes a few seconds it will generate hundreds of samples. We use the package **profvis** to visualize aggregates. 

Ways to use profvis

* From the Profile menu in RStudio
* With profvis::profvis().

Here are some of the functions the class of Introduction for Data Science of the year 2020 did for an assignment. These functions had to calculate the share of coincidence of votes between two countries after a given year.

These are the packages needed for the functions to properly work.


```r
#Packages specifically for the example functions to work properly
library(tidyverse)
library(knitr)
library(dplyr)
library(countrycode)
library(tidyr)
library(lubridate)
library(rmarkdown)
library(unvotes)
```


<details><summary>Click for syntax of function 1</summary>


```r
# even code
## Rodrigo's function
votes_agreement_calculator_1 <- function(country_code_1, country_code_2, year_min) {
  
  # get the data
  un_votes <- unvotes::un_votes
  un_votes_rc <- unvotes::un_roll_calls
  
  # for debuggin purpouses only
  # print(paste(country_code_1, country_code_2, year_min))
  
  # make sure the arguments are right
  # there's no voting before the UN exists in 1945 nor after the current year
  if (!is.numeric(year_min) | year_min < 1945 | year_min > lubridate::year(Sys.Date())) {
    stop ("year_min: invalid value")
  }
  
  # check if the country_code exists 
  
  if (!country_code_1 %in% un_votes$country_code | 
      !country_code_2 %in% un_votes$country_code) {
    stop ("country_code: invalide value")
  }
  
  # passing the checkings, keep working
  
  # clean the rol call base
  un_votes_rc_clean <- un_votes_rc |> 
    dplyr::mutate(
      # get the year of each votation
      year = lubridate::year(date)
    ) |> 
    # the necessary columns only
    dplyr::select(rcid, session, year) 
  
  # merge votes with rol call (to get year)
  un_votes_merged <- un_votes |> 
    # joining with roll call database cleaned
    dplyr::left_join(un_votes_rc_clean, by = "rcid") |> 
    dplyr::filter(year >= year_min & 
                    country_code %in% c(country_code_1, country_code_2)) 
  
  ## check if there is any country with 0 rows
  # if there aren't 2 countries, return NA
  # I did this because some country_codes were breking the code, and I prefer to do this check inside the function to make it more robust
  
  count_countries <- unique(un_votes_merged$country_code) |> 
    length() # how many countries are there?
  
  if(count_countries != 2) {
    message(paste("Warning:", country_code_1, "and", country_code_2, "since",
                  year_min, "has", count_countries, "values. It must have 2. Skiping this country."))
    return(NA)
  }
  
  
  # pivot to be able to compare country_code_1 and country_code_2
  votes_comparable <- un_votes_merged |> 
    dplyr::select(-session, -country) |> 
    tidyr::pivot_wider(names_from = country_code, 
                       values_from = vote) |> 
    # rename columns to make it easier to generalize
    dplyr::rename("country_1" = 3, "country_2" = 4) |> 
    dplyr::filter(!is.na(country_1) & !is.na(country_2))
  
  # group_by year, then count how many lines country_1 == country_2
  result <- votes_comparable |> 
    dplyr::mutate(
      equal = country_1 == country_2
    ) |> 
    dplyr::group_by(year) |> 
    dplyr::summarise(.groups = "drop", # dropping to do another summarise
                     agreed = sum(equal)  
    ) |> 
    dplyr::summarise(
      mean(agreed, na.rm = TRUE)
    ) |> 
    dplyr::pull(1) |> 
    as.numeric() |> 
    round(2)
  
  return(result)
  
}
```

</details>


<details><summary>Click for syntax of function 2</summary>


```r
#Alvaro's function

votes_agreement_calculator_2 <- function(country1, country2, year_min = "2000-01-01"){
  join <- left_join(un_votes, un_roll_calls, by = "rcid")
  c1 <- join %>% 
    filter(date >= year_min, country_code %in% country1) %>% 
    select(rcid, country, country_code, vote, session, date) %>% 
    rename(c1vote = vote)
  c2 <- join %>% 
    filter(date >= year_min, country_code %in% country2) %>% 
    select(rcid, country, country_code, vote, session, date) %>% 
    rename(c2vote = vote)
  c1c2 <- full_join(c1,c2, by="rcid")
  c1c2 <- c1c2 %>% 
    mutate(agreed = if_else(c1cote == c2vote,1,0)) %>% 
    filter(!is.na(agreed))
  total <- count(c1c2, agreed)
  c1c2ag <- total[2,2]
  return(round(c1c2ag/sum(total["n"],digits = 2)*100))
}
```

</details>

<details><summary>Click for syntax of function 3</summary>

```r
#Ben's function

# Create votes_agreement_calculator function
votes_agreement_calculator_3 <- function(ccode_a, ccode_b, year_min){
  
  # Create joined data set with necessary columns; create year column from date column; select relevant variables; discard NAs in country_code, filter rows for country a and country b
  un <- un_votes %>% 
    left_join(un_roll_calls, by = "rcid") %>% 
    mutate(year = format(date,"%Y"), date = NULL) %>% 
    select(rcid, country_code, vote, year) %>% 
    filter(!is.na(country_code)) %>%
    filter(country_code %in% c(ccode_a, ccode_b))
  
  # Pivot countries as columns and take values from the respective voting decision
  un_votes_wide <- un %>% 
    pivot_wider(names_from = "country_code", values_from = "vote")
  
  # Calculate agreement_share between ccode_a and ccode_b b since year_min
  agreement_share <- un_votes_wide %>%
    mutate(agreement = un_votes_wide[ccode_a] == un_votes_wide[ccode_b]) %>% #add agreement column
    filter(year >= year_min, !is.na(agreement)) %>% #discard NAs in agreement variable
    summarise(agreement_share = mean(agreement))%>% #calculate mean() of agreement variable
    as.numeric() #convert to numeric
  
  return(agreement_share)
}

# (a) Agreement rate for the United States and Russia for votes cast in 2000 and later
#votes_agreement_calculator_3("US", "RU", 2000)
```
</details>

<details><summary>Click for syntax of function 4</summary>

```r
#Luke's function

votes_agreement_calculator_4 <- function(country_1, country_2, year_min) {
  #Here we pull in the necessary data frames directly from the 'unvotes' package, assuming that the above code may not have been run yet
  un_joined_func <- un_votes %>%
    inner_join(un_roll_calls, by = "rcid") %>%
    group_by(year = year(date), country)
  #Here we create a data frame specifically for country_1 and rename their "vote" column accordingly
  df_1 <- un_joined_func %>%
    filter(country %in% country_1, year >= year_min) %>%
    rename(country_1_vote = vote)
  #We create a second data frame here for country_2 that does the same as above
  df_2 <- un_joined_func %>%
    filter(country %in% country_2, year >= year_min) %>%
    rename(country_2_vote = vote)
  #Now, we merge these two new data frames into one so that we can compare their newly named vote columns
    joined_dfs <- df_1 %>%
    inner_join(df_2, by = "rcid")
  #Below, we count the number of times that the two countries agree with each other in the UN
  country_vote_agreement <- count(joined_dfs, country_1_vote == country_2_vote)
  #Now we take the average and round it down to three decimal places
  average_country_vote_agreement <-   round((country_vote_agreement$n[2]/(country_vote_agreement$n[1]+country_vote_agreement$n[2])), digits=3)
  #Lastly, we create a list including the two countries' names and how often they voted with each other
  returned_list <- c(country_1, country_2, average_country_vote_agreement)
  return(average_country_vote_agreement)
}
```

</details>



```r
profvis({
  votes_agreement_calculator_1("US", "RU", 2000)
  votes_agreement_calculator_2("US", "RU", 2000)
  votes_agreement_calculator_3("US","RU","2000-01-01")
  votes_agreement_calculator_4("Russia", "United States", 2000)
})
```

```{=html}
<div id="htmlwidget-e2b4874b8e0690c0b630" style="width:100%;height:600px;" class="profvis html-widget"></div>
<script type="application/json" data-for="htmlwidget-e2b4874b8e0690c0b630">{"x":{"message":{"prof":{"time":[1,1,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,5,5,6,6,7,7,8,8,9,9,9,10,10,10,11,11,11,12,12,12,13,13,13,14,14,14,15,15,15,16,16,17,17,18,18,19,19,20,20,21,21,22,22,23,23,24,24,25,25,26,26,27,27,28,28,29,29,30,30,30,30,31,31,31,32,32,33,33,34,34,34,34,34,34,35,35,35,35,35,35,36,36,36,36,36,37,37,37,37,37,37,38,38,38,38,38,39,39,39,39,39,39,40,40,40,40,40,40,40,41,41,41,41,41,42,42,42,42,42,42,43,43,43,43,43,43,44,44,44,44,44,44,45,45,45,45,45,45,46,46,46,46,46,46,47,47,47,47,47,47,48,48,48,48,48,48,49,49,49,49,49,49,50,50,50,50,50,50,51,51,51,51,51,51,52,52,52,52,52,52,52,52,52,52,52,52,52,52,52,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,53,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,54,55,55,55,55,55,55,55,55,55,55,55,55,55,55,55,55,55,55,55,55,55,56,56,56,56,56,56,56,56,56,56,56,56,56,56,56,56,56,56,57,57,58,58,59,59,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,61,61,61,61,61,61,61,61,61,61,62,62,62,62,62,62,62,62,62,62,63,63,63,63,63,63,64,64,64,64,64,64,64,65,65,65,66,66,66,66,66,66,66,66,66,66,66,66,66,66,66,66,66,66,66,66,66,66,66,66,66,66,66,67,67,67,67,67,67,67,67,67,67,67,67,67,67,68,68,68,68,68,68,69,69,69,69,69,69,70,70,70,70,70,70,71,71,71,71,71,72,72,72,72,72,72,73,73,73,73,73,73,74,74,74,74,74,75,75,75,75,75,75,75,76,76,76,76,76,77,77,77,77,77,78,78,78,78,78,79,79,79,79,79,79,80,80,80,80,80,81,81,81,81,81,81,82,82,82,82,82,82,83,83,83,83,83,83,84,84,84,84,84,84,85,85,85,85,85,85,86,86,86,86,86,86,87,87,87,87,87,87,88,88,88,88,88,88,89,89,89,89,89,89,90,90,90,90,90,90,91,91,91,91,91,91,92,92,92,92,92,92,93,93,93,93,93,93,94,94,94,94,94,95,95,95,95,95,96,96,96,96,96,97,97,97,97,97,97,97,97,97,97,97,97,97,98,98,99,99,100,100,101,101,101,101,101,101,101,102,102,102,102,102,102,102,102,103,103,103,103,103,103,103,103,103,104,104,105,105,106,106,107,107,108,108,108,109,109,109,109,110,110,110,110,110,110,110,110,111,111,112,112,112,112,112,112,112,112,112,112,112,112,112,112,112,112,112,112,112,112,112,112,112,112,112,113,113,113,113,113,113,113,113,113,113,113,113,113,113,113,113,113,113,113,113,113,113,113,113,113,114,114,114,114,114,114,114,114,114,114,114,114,114,114,114,114,114,114,114,114,114,114,114,114,114,115,115,115,115,115,115,115,115,115,115,115,115,115,115,115,115,115,115,115,115,115,115,115,115,115,116,116,116,116,116,116,116,116,116,116,116,116,116,116,116,116,116,116,116,116,116,116,116,116,116,117,117,117,117,117,117,117,117,117,117,117,117,117,117,117,117,117,117,117,117,117,117,117,117,117,118,118,118,118,118,118,118,118,118,118,118,118,118,118,118,118,118,118,118,118,118,118,118,118,118,119,119,119,119,119,119,119,119,119,119,119,119,119,119,119,119,119,119,119,119,119,119,119,119,119,120,120,120,120,120,120,120,120,120,120,120,120,120,120,120,120,120,120,120,120,120,120,120,120,120,120,121,121,121,121,121,121,121,121,121,121,121,121,121,121,121,121,121,121,121,121,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,122,123,123,123,123,123,123,123,123,123,123,123,123,123,123,123,123,123,123,123,123,124,124,124,124,124,124,124,124,124,124,124,124,124,124,124,124,124,124,124,124,125,125,125,125,125,125,125,125,125,125,125,125,125,125,125,125,125,125,125,125,125,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,126,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,127,128,128,128,128,128,128,128,128,128,128,128,128,128,128,128,128,128,128,128,128,128,129,129,129,129,129,129,129,129,129,129,129,129,129],"depth":[2,1,28,27,26,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,2,1,2,1,2,1,2,1,3,2,1,3,2,1,3,2,1,3,2,1,3,2,1,3,2,1,3,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,2,1,4,3,2,1,3,2,1,2,1,2,1,6,5,4,3,2,1,6,5,4,3,2,1,5,4,3,2,1,6,5,4,3,2,1,5,4,3,2,1,6,5,4,3,2,1,7,6,5,4,3,2,1,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,29,28,27,26,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,41,40,39,38,37,36,35,34,33,32,31,30,29,28,27,26,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,2,1,2,1,2,1,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,10,9,8,7,6,5,4,3,2,1,10,9,8,7,6,5,4,3,2,1,6,5,4,3,2,1,7,6,5,4,3,2,1,3,2,1,27,26,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,14,13,12,11,10,9,8,7,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,5,4,3,2,1,7,6,5,4,3,2,1,5,4,3,2,1,5,4,3,2,1,5,4,3,2,1,6,5,4,3,2,1,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,6,5,4,3,2,1,5,4,3,2,1,5,4,3,2,1,5,4,3,2,1,13,12,11,10,9,8,7,6,5,4,3,2,1,2,1,2,1,2,1,7,6,5,4,3,2,1,8,7,6,5,4,3,2,1,9,8,7,6,5,4,3,2,1,2,1,2,1,2,1,2,1,3,2,1,4,3,2,1,8,7,6,5,4,3,2,1,2,1,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,26,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,29,28,27,26,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,30,29,28,27,26,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,30,29,28,27,26,25,24,23,22,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,21,20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1,13,12,11,10,9,8,7,6,5,4,3,2,1],"label":["invisible","rmarkdown::render","make.codeBuf","genCode","cmpCallArgs","cmpCallSymFun","cmpCall","cmp","cmpPrim1","h","tryInline","cmpCall","cmp","cmpPrim2","h","tryInline","cmpCall","cmp","h","tryInline","cmpCall","cmp","h","tryInline","cmpCall","cmp","genCode","cmpfun","compiler:::tryCmpfun","rmarkdown::render","putconst","cb$putcode","cmpCallExprFun","cmpCall","cmp","genCode","cmpCallArgs","cmpCallExprFun","cmpCall","cmp","cmpSymbolAssign","h","tryInline","cmpCall","cmp","h","tryInline","cmpCall","cmp","genCode","cmpfun","compiler:::tryCmpfun","rmarkdown::render","cmpCall","cmp","cmpCallExprFun","cmpCall","cmp","genCode","cmpCallArgs","cmpCallExprFun","cmpCall","cmp","cmpSymbolAssign","h","tryInline","cmpCall","cmp","h","tryInline","cmpCall","cmp","genCode","cmpfun","compiler:::tryCmpfun","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","<GC>","lazyLoadDBfetch","rmarkdown::render","<GC>","lazyLoadDBfetch","rmarkdown::render","<GC>","lazyLoadDBfetch","rmarkdown::render","<GC>","lazyLoadDBfetch","rmarkdown::render","<GC>","lazyLoadDBfetch","rmarkdown::render","<GC>","lazyLoadDBfetch","rmarkdown::render","<GC>","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","<GC>","%in%","votes_agreement_calculator_1","rmarkdown::render","%in%","votes_agreement_calculator_1","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","vec_match","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","vec_match","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","lengths","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","<GC>","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","unlist","index_flatten","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_1","rmarkdown::render","file.exists","packageDescription","utils::packageVersion",".rlang_cli_has_cli","has_ansi","new_quo_deparser","expr_deparse","infix_overflows","FUN","vapply",".rlang_purrr_map_mold","map_chr","exprs_auto_name","rlang:::quos_auto_name","rmarkdown::render","get","getSetterInlineHandler","trySetterInline","cmpSetterCall","cmpComplexAssign","h","tryInline","cmpCall","cmp","h","tryInline","cmpCall","cmp","h","tryInline","cmpCall","cmp","h","tryInline","cmpCall","cmp","h","tryInline","cmpCall","cmp","genCode","cmpfun","compiler:::tryCmpfun","rmarkdown::render","caller_env","%<~%","env_get","$.r6lite","method","lines$get_lines","sexp_deparse","deparser","method","lines$deparse","args_deparse","deparser","sexp_deparse","deparser","method","lines$deparse","operand_deparse","binary_op_deparse","deparser","op_deparse","deparser","sexp_deparse","deparser","method","lines$deparse","operand_deparse","binary_op_deparse","deparser","op_deparse","deparser","sexp_deparse","quo_deparse","expr_deparse","infix_overflows","FUN","vapply",".rlang_purrr_map_mold","map_chr","exprs_auto_name","rlang:::quos_auto_name","rmarkdown::render","rlang::is_environment","check_environment","env_get","$.r6lite","method","lines$get_lines","binary_op_deparse","deparser","op_deparse","deparser","sexp_deparse","quo_deparse","expr_deparse","as_label_infix","FUN","vapply",".rlang_purrr_map_mold","map_chr","exprs_auto_name","rlang:::quos_auto_name","rmarkdown::render","expr_interp","$.r6lite","operand_deparse","binary_op_deparse","deparser","op_deparse","deparser","sexp_deparse","quo_deparse","expr_deparse","as_label_infix","FUN","vapply",".rlang_purrr_map_mold","map_chr","exprs_auto_name","rlang:::quos_auto_name","rmarkdown::render",">=","rmarkdown::render","%in%","rmarkdown::render",".Call","rmarkdown::render","isTRUE","knitr_in_progress","in_knitr","exit_frame","add_handler","defer","local_bindings","walk_data_tree","reduce_sels","eval_c","walk_data_tree","vars_select_eval","eval_select_impl","tidyselect::eval_select","select.data.frame","votes_agreement_calculator_1","rmarkdown::render","withCallingHandlers","try_fetch","with_entraced_errors","with_subscript_errors","eval_select_impl","tidyselect::eval_select","build_wider_id_cols_expr","pivot_wider.data.frame","votes_agreement_calculator_1","rmarkdown::render","vapply",".rlang_purrr_map_mold","map_lgl","endots","enquos","dplyr_quosures","filter_rows","filter.data.frame","votes_agreement_calculator_1","rmarkdown::render","new_dplyr_quosure","dplyr_quosures","filter_rows","filter.data.frame","votes_agreement_calculator_1","rmarkdown::render","vec_group_loc","vec_split_id_order","compute_groups","grouped_df","group_by.data.frame","votes_agreement_calculator_1","rmarkdown::render","summarise.data.frame","votes_agreement_calculator_1","rmarkdown::render","getInlineInfo","tryInline","cmpCall","cmp","genCode","cmpCallArgs","cmpCallSymFun","cmpCall","cmp","genCode","cmpCallArgs","cmpCallSymFun","cmpCall","cmp","cmpSymbolAssign","h","tryInline","cmpCall","cmp","h","tryInline","cmpCall","cmp","genCode","cmpfun","compiler:::tryCmpfun","rmarkdown::render","exists","findCenvVar","getInlineInfo","tryInline","cmpCall","cmp","h","tryInline","cmpCall","cmp","genCode","cmpfun","compiler:::tryCmpfun","rmarkdown::render","vec_match","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","vec_match","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","vec_match","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","lengths","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","unlist","index_flatten","join_rows","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","<GC>","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","vec_slice","join_mutate","left_join.data.frame","votes_agreement_calculator_2","rmarkdown::render","Sys.getenv","cli::num_ansi_colors","has_ansi","new_quo_deparser","expr_deparse","infix_overflows","FUN","vapply",".rlang_purrr_map_mold","map_chr","exprs_auto_name","rlang:::quos_auto_name","rmarkdown::render","NextMethod","rmarkdown::render","%in%","rmarkdown::render",".Call","rmarkdown::render","dots_split","env","vars_select_eval","eval_select_impl","tidyselect::eval_select","select.data.frame","rmarkdown::render","quo_squash","FUN","vapply",".rlang_purrr_map_mold","map_chr","exprs_auto_name","rlang:::quos_auto_name","rmarkdown::render","duplicated.default","base::setdiff","setdiff.default","group_vars.data.frame","initialize","DataMask$new","filter_rows","filter.data.frame","rmarkdown::render","NextMethod","rmarkdown::render","%in%","rmarkdown::render","%in%","rmarkdown::render","%in%","rmarkdown::render","<GC>","%in%","rmarkdown::render","vec_slice","dplyr_row_slice.data.frame","filter.data.frame","rmarkdown::render","eval_c","walk_data_tree","vars_select_eval","eval_select_impl","rename_impl","tidyselect::eval_rename","rename.data.frame","rmarkdown::render","lazyLoadDBfetch","rmarkdown::render","Sys.sleep","callRemote","callFun","rstudioapi::getThemeInfo","get_rstudio_theme","detect_dark_theme","builtin_theme","clii_init","app$new","cliapp","start_app","cli__fmt","cli_fmt","cli_format",".rlang_cli_format","cli_format","cnd_message_format","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","Sys.sleep","callRemote","callFun","rstudioapi::getThemeInfo","get_rstudio_theme","detect_dark_theme","builtin_theme","clii_init","app$new","cliapp","start_app","cli__fmt","cli_fmt","cli_format",".rlang_cli_format","cli_format","cnd_message_format","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","Sys.sleep","callRemote","callFun","rstudioapi::getThemeInfo","get_rstudio_theme","detect_dark_theme","builtin_theme","clii_init","app$new","cliapp","start_app","cli__fmt","cli_fmt","cli_format",".rlang_cli_format","cli_format","cnd_message_format","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","Sys.sleep","callRemote","callFun","rstudioapi::getThemeInfo","get_rstudio_theme","detect_dark_theme","builtin_theme","clii_init","app$new","cliapp","start_app","cli__fmt","cli_fmt","cli_format",".rlang_cli_format","cli_format","cnd_message_format","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","Sys.sleep","callRemote","callFun","rstudioapi::getThemeInfo","get_rstudio_theme","detect_dark_theme","builtin_theme","clii_init","app$new","cliapp","start_app","cli__fmt","cli_fmt","cli_format",".rlang_cli_format","cli_format","cnd_message_format","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","Sys.sleep","callRemote","callFun","rstudioapi::getThemeInfo","get_rstudio_theme","detect_dark_theme","builtin_theme","clii_init","app$new","cliapp","start_app","cli__fmt","cli_fmt","cli_format",".rlang_cli_format","cli_format","cnd_message_format","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","Sys.sleep","callRemote","callFun","rstudioapi::getThemeInfo","get_rstudio_theme","detect_dark_theme","builtin_theme","clii_init","app$new","cliapp","start_app","cli__fmt","cli_fmt","cli_format",".rlang_cli_format","cli_format","cnd_message_format","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","Sys.sleep","callRemote","callFun","rstudioapi::getThemeInfo","get_rstudio_theme","detect_dark_theme","builtin_theme","clii_init","app$new","cliapp","start_app","cli__fmt","cli_fmt","cli_format",".rlang_cli_format","cli_format","cnd_message_format","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","grepl","FUN","lapply","FUN","lapply","theme_create","clii_add_theme","app$add_theme","clii_init","app$new","cliapp","start_app","cli__fmt","cli_fmt","cli_format",".rlang_cli_format","cli_format","cnd_message_format","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","$","clii__container_start","FUN","lapply","clii_bullets","<Anonymous>","cli__fmt","cli_fmt","cli_format",".rlang_cli_format","cli_format","cnd_message_format","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","<Anonymous>","get_parentpid","detect_new","rstudio$detect","console_width","clii__get_width","app$get_width","clii__xtext","app$xtext","clii_text","app$text","FUN","lapply","clii_bullets","<Anonymous>","cli__fmt","cli_fmt","cli_format",".rlang_cli_format","cli_format","cnd_message_format","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","Sys.getenv","get_data","detect_new","rstudio$detect","rstudio_detect","get_rstudio_fg_color0","get_rstudio_fg_color","update_rstudio_color","cli_format",".rlang_cli_format","cli_format","cnd_message_format","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","<Anonymous>","get_parentpid","detect_new","rstudio$detect","rstudio_detect","get_rstudio_fg_color0","get_rstudio_fg_color","update_rstudio_color","cli_format",".rlang_cli_format","cli_format","cnd_message_format","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","%in%","match_selector_node","match_selector","clii__container_start","clii_div","<Anonymous>","cli__fmt","cli_fmt","cli::format_message",".rlang_cli_format_inline","format_code","format_error_call","cnd_message_format_prefixed","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","%in%","get_parentpid","detect_new","rstudio$detect","console_width","clii__get_width","app$get_width","clii__xtext","app$xtext","clii_text","app$text","FUN","lapply","clii_bullets","<Anonymous>","cli__fmt","cli_fmt","cli::format_message",".rlang_cli_format_inline","format_code","format_error_call","cnd_message_format_prefixed","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","<Anonymous>","get_parentpid","detect_new","rstudio$detect","console_width","clii__get_width","app$get_width","clii__xtext","app$xtext","clii_text","app$text","FUN","lapply","clii_bullets","<Anonymous>","cli__fmt","cli_fmt","cli::format_message",".rlang_cli_format_inline","format_code","format_error_call","cnd_message_format_prefixed","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","<Anonymous>","get_parentpid","detect_new","rstudio$detect","rstudio_detect","get_rstudio_fg_color0","get_rstudio_fg_color","update_rstudio_color","cli::format_message",".rlang_cli_format_inline","format_code","format_error_call","cnd_message_format_prefixed","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render","[[","cnd_footer","cnd_message_lines","cnd_message_format","cnd_message_format_prefixed","cnd_message","conditionMessage.rlang_error","signalCondition","signal_abort","abort","h",".handleSimpleError","rmarkdown::render"],"filenum":[null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,1,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,1,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,1,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,1,null,null,1,null,null,null,null,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,null,1,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,1,null,null,null,null,null,null,null,null,null,1,null,null,null,null,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,null,1,null,null,1,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,1,null,null,null,null,null,null,null,null,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,1,null,null,null,null,null,null,1,null,null,null,null,1,null,null,null,null,1,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,null,1,null,null,null,null,1,null,null,null,null,1,null,null,null,null,1,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null],"linenum":[null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,3,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,3,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,3,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,3,null,null,3,null,null,null,null,null,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,null,null,3,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,3,null,null,null,null,null,null,null,null,null,3,null,null,null,null,null,null,null,null,null,3,null,null,null,null,null,3,null,null,null,null,null,null,3,null,null,3,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,4,null,null,null,null,null,null,null,null,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,4,null,null,null,null,null,null,4,null,null,null,null,4,null,null,null,null,4,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,null,4,null,null,null,null,4,null,null,null,null,4,null,null,null,null,4,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null],"memalloc":[14.3191070556641,14.3191070556641,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.5503921508789,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.6628189086914,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,14.7739944458008,47.1437911987305,47.1437911987305,47.1437911987305,47.1437911987305,47.1437911987305,47.1437911987305,47.1437911987305,47.1437911987305,46.5954284667969,46.5954284667969,46.5954284667969,46.5952301025391,46.5952301025391,46.5952301025391,46.5952301025391,46.5952301025391,46.5952301025391,46.5952301025391,46.5952301025391,46.5952301025391,46.5952301025391,46.5952301025391,46.5952301025391,46.5952301025391,46.5952301025391,46.5952301025391,46.5950393676758,46.5950393676758,46.5950393676758,78.4208984375,78.4208984375,91.6967468261719,91.6967468261719,91.6979370117188,91.6979370117188,91.698112487793,91.698112487793,91.6981353759766,91.6981353759766,91.6985855102539,91.6985855102539,91.6986389160156,91.6986389160156,91.6986541748047,91.6986541748047,98.3367462158203,98.3367462158203,98.3369064331055,98.3369064331055,98.337158203125,98.337158203125,98.3371734619141,98.3371734619141,101.655723571777,101.655723571777,105.501617431641,105.501617431641,106.538017272949,106.538017272949,106.538017272949,106.538017272949,84.502685546875,84.502685546875,84.502685546875,85.8244476318359,85.8244476318359,87.1412124633789,87.1412124633789,91.5047836303711,91.5047836303711,91.5047836303711,91.5047836303711,91.5047836303711,91.5047836303711,94.8233337402344,94.8233337402344,94.8233337402344,94.8233337402344,94.8233337402344,94.8233337402344,101.460433959961,101.460433959961,101.460433959961,101.460433959961,101.460433959961,104.779052734375,104.779052734375,104.779052734375,104.779052734375,104.779052734375,104.779052734375,111.416152954102,111.416152954102,111.416152954102,111.416152954102,111.416152954102,111.416152954102,111.416152954102,111.416152954102,111.416152954102,111.416152954102,111.416152954102,85.2156677246094,85.2156677246094,85.2156677246094,85.2156677246094,85.2156677246094,85.2156677246094,85.2156677246094,101.809837341309,101.809837341309,101.809837341309,101.809837341309,101.809837341309,108.447036743164,108.447036743164,108.447036743164,108.447036743164,108.447036743164,108.447036743164,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,108.325004577637,76.4991455078125,76.4991455078125,76.4991455078125,76.4991455078125,76.4991455078125,76.4991455078125,83.1482162475586,83.1482162475586,83.1482162475586,83.1482162475586,83.1482162475586,83.1482162475586,83.1482162475586,83.1482162475586,83.1482162475586,83.1482162475586,83.1482162475586,83.1482162475586,83.1482162475586,83.1482162475586,83.1482162475586,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.6202239990234,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.7573165893555,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.8681182861328,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,83.89892578125,93.9939422607422,93.9939422607422,103.949638366699,103.949638366699,113.905311584473,113.905311584473,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.156326293945,114.655258178711,114.655258178711,114.655258178711,114.655258178711,114.655258178711,114.655258178711,114.655258178711,114.655258178711,114.655258178711,114.655258178711,115.397880554199,115.397880554199,115.397880554199,115.397880554199,115.397880554199,115.397880554199,115.397880554199,115.397880554199,115.397880554199,115.397880554199,115.462158203125,115.462158203125,115.462158203125,115.462158203125,115.462158203125,115.462158203125,115.901679992676,115.901679992676,115.901679992676,115.901679992676,115.901679992676,115.901679992676,115.901679992676,116.332763671875,116.332763671875,116.332763671875,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.652282714844,116.831359863281,116.831359863281,116.831359863281,116.831359863281,116.831359863281,116.831359863281,116.831359863281,116.831359863281,116.831359863281,116.831359863281,116.831359863281,116.831359863281,116.831359863281,116.831359863281,120.620269775391,120.620269775391,120.620269775391,120.620269775391,120.620269775391,120.620269775391,123.938819885254,123.938819885254,123.938819885254,123.938819885254,123.938819885254,123.938819885254,123.938819885254,123.938819885254,123.938819885254,123.938819885254,123.938819885254,123.938819885254,130.57592010498,130.57592010498,130.57592010498,130.57592010498,130.57592010498,133.894538879395,133.894538879395,133.894538879395,133.894538879395,133.894538879395,133.894538879395,123.123931884766,123.123931884766,123.123931884766,123.123931884766,123.123931884766,123.123931884766,103.401336669922,103.401336669922,103.401336669922,103.401336669922,103.401336669922,103.401336669922,103.401336669922,103.401336669922,103.401336669922,103.401336669922,103.401336669922,103.401336669922,113.358726501465,113.358726501465,113.358726501465,113.358726501465,113.358726501465,126.633018493652,126.633018493652,126.633018493652,126.633018493652,126.633018493652,126.633018493652,126.633018493652,126.633018493652,126.633018493652,126.633018493652,129.951835632324,129.951835632324,129.951835632324,129.951835632324,129.951835632324,129.951835632324,119.875923156738,119.875923156738,119.875923156738,119.875923156738,119.875923156738,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,129.831726074219,86.4419021606445,86.4419021606445,86.4419021606445,86.4419021606445,86.4419021606445,86.4419021606445,93.0789947509766,93.0789947509766,93.0789947509766,93.0789947509766,93.0789947509766,106.35334777832,106.35334777832,106.35334777832,106.35334777832,106.35334777832,112.990493774414,112.990493774414,112.990493774414,112.990493774414,112.990493774414,113.014007568359,113.014007568359,113.014007568359,113.014007568359,113.014007568359,113.014007568359,113.014007568359,113.014007568359,113.014007568359,113.014007568359,113.014007568359,113.014007568359,113.014007568359,122.999069213867,122.999069213867,129.636207580566,129.636207580566,136.273330688477,136.273330688477,139.961349487305,139.961349487305,139.961349487305,139.961349487305,139.961349487305,139.961349487305,139.961349487305,140.065299987793,140.065299987793,140.065299987793,140.065299987793,140.065299987793,140.065299987793,140.065299987793,140.065299987793,140.102516174316,140.102516174316,140.102516174316,140.102516174316,140.102516174316,140.102516174316,140.102516174316,140.102516174316,140.102516174316,150.060737609863,150.060737609863,156.697860717773,156.697860717773,160.016426086426,160.016426086426,160.016426086426,160.016426086426,160.01643371582,160.01643371582,160.01643371582,129.77855682373,129.77855682373,129.77855682373,129.77855682373,130.23755645752,130.23755645752,130.23755645752,130.23755645752,130.23755645752,130.23755645752,130.23755645752,130.23755645752,131.782279968262,131.782279968262,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,132.860275268555,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.054107666016,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.308807373047,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.60400390625,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.811882019043,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.814399719238,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,133.985191345215,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.233139038086,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.234367370605,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.255332946777,134.25626373291,134.25626373291,134.25626373291,134.25626373291,134.25626373291,134.25626373291,134.25626373291,134.25626373291,134.25626373291,134.25626373291,134.25626373291,134.25626373291,134.25626373291],"meminc":[0,0,0.231285095214844,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.1124267578125,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.111175537109375,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,32.3697967529297,0,0,0,0,0,0,0,-0.548362731933594,0,0,-0.0001983642578125,0,0,0,0,0,0,0,0,0,0,0,0,0,0,-0.00019073486328125,0,0,31.8258590698242,0,13.2758483886719,0,0.001190185546875,0,0.00017547607421875,0,2.288818359375e-05,0,0.00045013427734375,0,5.340576171875e-05,0,1.52587890625e-05,0,6.63809204101562,0,0.00016021728515625,0,0.00025177001953125,0,1.52587890625e-05,0,3.31855010986328,0,3.84589385986328,0,1.03639984130859,0,0,0,-22.0353317260742,0,0,1.32176208496094,0,1.31676483154297,0,4.36357116699219,0,0,0,0,0,3.31855010986328,0,0,0,0,0,6.63710021972656,0,0,0,0,3.31861877441406,0,0,0,0,0,6.63710021972656,0,0,0,0,0,0,0,0,0,0,-26.2004852294922,0,0,0,0,0,0,16.5941696166992,0,0,0,0,6.63719940185547,0,0,0,0,0,-0.122032165527344,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,-31.8258590698242,0,0,0,0,0,6.64907073974609,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.472007751464844,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.137092590332031,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.110801696777344,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.0308074951171875,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,10.0950164794922,0,9.95569610595703,0,9.95567321777344,0,0.251014709472656,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.498931884765625,0,0,0,0,0,0,0,0,0,0.742622375488281,0,0,0,0,0,0,0,0,0,0.0642776489257812,0,0,0,0,0,0.439521789550781,0,0,0,0,0,0,0.431083679199219,0,0,0.31951904296875,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.1790771484375,0,0,0,0,0,0,0,0,0,0,0,0,0,3.78890991210938,0,0,0,0,0,3.31855010986328,0,0,0,0,0,0,0,0,0,0,0,6.63710021972656,0,0,0,0,3.31861877441406,0,0,0,0,0,-10.7706069946289,0,0,0,0,0,-19.7225952148438,0,0,0,0,0,0,0,0,0,0,0,9.95738983154297,0,0,0,0,13.2742919921875,0,0,0,0,0,0,0,0,0,3.31881713867188,0,0,0,0,0,-10.0759124755859,0,0,0,0,9.95580291748047,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,-43.3898239135742,0,0,0,0,0,6.63709259033203,0,0,0,0,13.2743530273438,0,0,0,0,6.63714599609375,0,0,0,0,0.0235137939453125,0,0,0,0,0,0,0,0,0,0,0,0,9.98506164550781,0,6.63713836669922,0,6.63712310791016,0,3.68801879882812,0,0,0,0,0,0,0.103950500488281,0,0,0,0,0,0,0,0.0372161865234375,0,0,0,0,0,0,0,0,9.95822143554688,0,6.63712310791016,0,3.31856536865234,0,0,0,7.62939453125e-06,0,0,-30.2378768920898,0,0,0,0.458999633789062,0,0,0,0,0,0,0,1.54472351074219,0,1.07799530029297,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.193832397460938,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.25469970703125,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.295196533203125,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.207878112792969,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.0025177001953125,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.170791625976562,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.247947692871094,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.00122833251953125,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.020965576171875,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.0009307861328125,0,0,0,0,0,0,0,0,0,0,0,0],"filename":[null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,"<expr>",null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,"<expr>",null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,"<expr>",null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,"<expr>",null,null,"<expr>",null,null,null,null,null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,null,"<expr>",null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,"<expr>",null,null,null,null,null,null,null,null,null,"<expr>",null,null,null,null,null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,null,"<expr>",null,null,"<expr>",null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,"<expr>",null,null,null,null,null,null,null,null,null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,"<expr>",null,null,null,null,null,null,"<expr>",null,null,null,null,"<expr>",null,null,null,null,"<expr>",null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,null,"<expr>",null,null,null,null,"<expr>",null,null,null,null,"<expr>",null,null,null,null,"<expr>",null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null]},"interval":10,"files":[{"filename":"<expr>","content":"\nprofvis({\n  votes_agreement_calculator_1(\"US\", \"RU\", 2000)\n  votes_agreement_calculator_2(\"US\", \"RU\", 2000)\n  votes_agreement_calculator_3(\"US\",\"RU\",\"2000-01-01\")\n  votes_agreement_calculator_4(\"Russia\", \"United States\", 2000)\n})","normpath":"<expr>"}],"prof_output":"C:\\Users\\carme\\AppData\\Local\\Temp\\RtmpUZysXt\\file6b081b656c4e.prof","highlight":{"output":["^output\\$"],"gc":["^<GC>$"],"stacktrace":["^\\.\\.stacktraceo(n|ff)\\.\\.$"]},"split":"h"}},"evals":[],"jsHooks":[]}</script>
```

Vis output shows the source code, overlaid with bar graphs for memory and execution time for each line of code. 

Best practice: store code in a separate file and source()ing it in; this will ensure you get the best connection between profiling data and source code.

The bottom pane displays a flame graph showing the full call stack. It allows to see when objects are called more than one time. This is also displayed in the data tab, which let's you zoom interactively.

R will give this output once we run the profiling function. On the top panel, we can see the memory allocated to each function. In this case, two of the functions are significantly higher in memory and time than the two others. In the bottom panel, we can see the elements of the functions, the time they took to run, and the garbage collector in grey.

## Memory profiling

The special entry <GC> indicates that the garbage collector is running. If it's taking a lot of time, it means there are a lot of short-lived objects. When this happens, check the memory column and look which lines are taking a lot of memory(the bar on the right) and freed(the bar on the left).

**Limitations**

* Profiling doesn't extend to C code. You can see if your R code calls C/C++ code but not what functions are called inside of your C/C++ code.
* If there are many anonymous functions, it is hard to identify which is the function being called. It is better to name the functions.
* R's profiler doesn't store enough information to disentangle lazy evaluation.

## Microbenching

A **microbenchmark** is a measurement of the performance of a very small piece of code. They are useful for comparing small snippets of code for specific tasks. Be very careful of generalizing: the observed differences in microbenchmarks will typically be dominated by higher-order effects in real code; a deep understanding of subatommics physics is not very helpful when baking.


```r
library(bench)
```

By default, bench::mark() runs each expression at least once (min_iterations = 1), and at most enough times to take 0.5 s (min_time = 0.5). It checks that each run returns the same value which is typically what you want microbenchmarking.

**Results**

bench::mark() returns the results as a tibble, with one row for each input expression, and the following columns:

* min, mean, median, mex, and itr/sec. These summarise the time taken by the expression. Focus on the minimum (best possible running time) and the median (the typical time).

* mem_alloc tells you the amount of memory allocated by the first run, and n_gc() tells you the total number of garbage collections over all runs. These are useful for assessing the memory usage of the expression.

* n_itr and total_time tells you how many times the expression was evaluated and how long that took in total. n_itr will always be greater than the min_iteration parameter, and total_time will always be greater than the min_time parameter

* result, memory, tie and gc are list-columns that store the raw underlying data

You can call them with []

**Interpreting results**

Pay attention to the units. It is useful to know how many times a function needs to run before it takes a second. If a microbenchmark takes:

* 1 ms, then one thousand calls take a second
* 1 micros, then one million calls take a second
* 1 nanos, then one billion calls take a second





# Improving Performance

The process of improving the performance of your code can be a daunting task initially as the solution can take many different potential forms. Four approaches to improving the performance of your code include:

- Code organization
- Checking for existing solutions
- Have your function do as little work as possible
- Vectorize your code


## Code Organization

One approach to improving the performance of your code is to compare the speed of different functions that accomplish the same goal. To illustrate this, the code below shows different functions that were utilized to show how often two countries voted with each other in the United Nations. We will see which of them is the most efficient.


```r
#First, we will install and load the necessary package, bench, which will allow us to compare the performance of small self-contained code chunks

#Package for measuring and improving the performance of code
library(bench)

#Packages specifically for the example functions to work properly
library(tidyverse)
library(knitr)
library(dplyr)
library(countrycode)
library(tidyr)
library(lubridate)
library(rmarkdown)
library(unvotes)


#Problem Set Functions

#Function #1
# Create votes_agreement_calculator function
function_1 <- function(ccode_a, ccode_b, year_min){
  
  # Create joined data set with necessary columns; create year column from date column; select relevant variables; discard NAs in country_code, filter rows for country a and country b
  un <- un_votes %>% 
    left_join(un_roll_calls, by = "rcid") %>% 
    mutate(year = format(date,"%Y"), date = NULL) %>% 
    select(rcid, country_code, vote, year) %>% 
    filter(!is.na(country_code)) %>%
    filter(country_code %in% c(ccode_a, ccode_b))
  
  # Pivot countries as columns and take values from the respective voting decision
  un_votes_wide <- un %>% 
    pivot_wider(names_from = "country_code", values_from = "vote")
  
  # Calculate agreement_share between ccode_a and ccode_b b since year_min
  agreement_share <- un_votes_wide %>%
    mutate(agreement = un_votes_wide[ccode_a] == un_votes_wide[ccode_b]) %>% #add agreement column
    filter(year >= year_min, !is.na(agreement)) %>% #discard NAs in agreement variable
    summarise(agreement_share = mean(agreement))%>% #calculate mean() of agreement variable
    as.numeric() #convert to numeric
  
  return(round(agreement_share, digits = 3))
}


#Function #2
function_2 <- function(country_1, country_2, year_min) {
  #Here we pull in the necessary data frames directly from the 'unvotes' package, assuming that the above code may not have been run yet
  un_joined_func <- un_votes %>%
    inner_join(un_roll_calls, by = "rcid") %>%
    group_by(year = year(date), country)
  #Here we create a data frame specifically for country_1 and rename their "vote" column accordingly
  df_1 <- un_joined_func %>%
    filter(country %in% country_1, year >= year_min) %>%
    rename(country_1_vote = vote)
  #We create a second data frame here for country_2 that does the same as above
  df_2 <- un_joined_func %>%
    filter(country %in% country_2, year >= year_min) %>%
    rename(country_2_vote = vote)
  #Now, we merge these two new data frames into one so that we can compare their newly named vote columns
    joined_dfs <- df_1 %>%
    inner_join(df_2, by = "rcid")
  #Below, we count the number of times that the two countries agree with each other in the UN
  country_vote_agreement <- count(joined_dfs, country_1_vote == country_2_vote)
  #Now we take the average and round it down to three decimal places
  average_country_vote_agreement <-   round((country_vote_agreement$n[2]/(country_vote_agreement$n[1]+country_vote_agreement$n[2])), digits=3)
  #Lastly, we create a list including the two countries' names and how often they voted with each other
  returned_list <- c(country_1, country_2, average_country_vote_agreement)
  return(average_country_vote_agreement)
}




#Here we will compare the functions provided above
bench::mark(
  function_1("US", "RU", 2000),
  function_2("Russia", "United States", 2000)
)
```

```
## # A tibble: 2 × 6
##   expression                                       min   median itr/se…¹ mem_a…²
##   <bch:expr>                                  <bch:tm> <bch:tm>    <dbl> <bch:b>
## 1 function_1("US", "RU", 2000)                   9.65s    9.65s    0.104   234MB
## 2 function_2("Russia", "United States", 2000)    2.21s    2.21s    0.451   242MB
## # … with 1 more variable: `gc/sec` <dbl>, and abbreviated variable names
## #   ¹​`itr/sec`, ²​mem_alloc
```

After observing the results, we can see that Function #2 takes less time to run than Function #1. Additionally, less memory is utilized by Function #1

## Checking for exisiting solutions

After your have exhausted your own solutions, it is helpful to consult outside resources to see if the issue has been encountered before since it very likely has been. Potential options could include:

- rseek at https://rseek.org/
- stackoverflow at https://stackoverflow.com/
- CRAN Task Views at https://cran.rstudio.com/web/views/
- Seeking input from fellow students, professors, colleagues, etc.

## Have your function do as little work as possible

When choosing (premade) functions, try to make it to as little work as possible. This means choosing a function which will accomplish your desired goal while using the least amount of effort. Intuitively knowing which functions are faster than others requires a high level of familiarity with R code. Some common items to note include:

- readr::read_csv() is significantly faster than read.csv()
- rowSums(), colSums(), rowMeans(), and colMeans() are faster than equivalent invocations that use apply() because they are vectorised


## Vectorize your code
Vectorizing your code can be a very powerful tool in reducing its run time. By utilizing vectorized functions, you can insure that the function is being applied to the entire vector at the same time, rather than one at a time as with a non-vectorized function. The example below illustrates this with an example that multiplies vectors.



```r
#Microbenchmark returns summaries from the test
library(microbenchmark)
```

```
## Warning: package 'microbenchmark' was built under R version 4.2.2
```

```r
#Function (vectorized) that multiplies two vectors. 
vectorized_vector <- function(n) {
    x <- 1:n
    y <- x * x
    return(y)
}

#Function (non-vectorized) that multiplies two vectors
vector_for_loop <- function(n) {
    x <- 1:n
    y <- vector("numeric", length = n)
    
    for (i in 1:n) {
        y[i] <- x[i] * x[i]
    }
    return(y)
}



microbenchmark(vectorized_vector(100),
               vector_for_loop(100),
               times = 100)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["expr"],"name":[1],"type":["fct"],"align":["left"]},{"label":["time"],"name":[2],"type":["dbl"],"align":["right"]}],"data":[{"1":"vector_for_loop(100)","2":"16311400"},{"1":"vector_for_loop(100)","2":"40600"},{"1":"vectorized_vector(100)","2":"32000"},{"1":"vector_for_loop(100)","2":"39300"},{"1":"vector_for_loop(100)","2":"40100"},{"1":"vector_for_loop(100)","2":"39400"},{"1":"vectorized_vector(100)","2":"6713900"},{"1":"vector_for_loop(100)","2":"47700"},{"1":"vector_for_loop(100)","2":"37800"},{"1":"vector_for_loop(100)","2":"37300"},{"1":"vectorized_vector(100)","2":"5100"},{"1":"vectorized_vector(100)","2":"3400"},{"1":"vector_for_loop(100)","2":"36900"},{"1":"vectorized_vector(100)","2":"3800"},{"1":"vectorized_vector(100)","2":"3400"},{"1":"vector_for_loop(100)","2":"37200"},{"1":"vector_for_loop(100)","2":"36700"},{"1":"vectorized_vector(100)","2":"4000"},{"1":"vector_for_loop(100)","2":"36700"},{"1":"vector_for_loop(100)","2":"36900"},{"1":"vector_for_loop(100)","2":"36600"},{"1":"vector_for_loop(100)","2":"36600"},{"1":"vectorized_vector(100)","2":"3700"},{"1":"vector_for_loop(100)","2":"38600"},{"1":"vectorized_vector(100)","2":"3400"},{"1":"vector_for_loop(100)","2":"39000"},{"1":"vector_for_loop(100)","2":"38700"},{"1":"vector_for_loop(100)","2":"38700"},{"1":"vectorized_vector(100)","2":"3600"},{"1":"vectorized_vector(100)","2":"3400"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vector_for_loop(100)","2":"39700"},{"1":"vector_for_loop(100)","2":"39300"},{"1":"vector_for_loop(100)","2":"49300"},{"1":"vectorized_vector(100)","2":"3700"},{"1":"vector_for_loop(100)","2":"58300"},{"1":"vector_for_loop(100)","2":"41800"},{"1":"vector_for_loop(100)","2":"48700"},{"1":"vector_for_loop(100)","2":"96000"},{"1":"vectorized_vector(100)","2":"33800"},{"1":"vectorized_vector(100)","2":"5700"},{"1":"vector_for_loop(100)","2":"59600"},{"1":"vectorized_vector(100)","2":"6200"},{"1":"vectorized_vector(100)","2":"5400"},{"1":"vectorized_vector(100)","2":"42700"},{"1":"vectorized_vector(100)","2":"4800"},{"1":"vectorized_vector(100)","2":"5000"},{"1":"vector_for_loop(100)","2":"62200"},{"1":"vector_for_loop(100)","2":"53100"},{"1":"vector_for_loop(100)","2":"51100"},{"1":"vectorized_vector(100)","2":"6300"},{"1":"vectorized_vector(100)","2":"5200"},{"1":"vector_for_loop(100)","2":"49200"},{"1":"vector_for_loop(100)","2":"46800"},{"1":"vectorized_vector(100)","2":"5000"},{"1":"vector_for_loop(100)","2":"46700"},{"1":"vectorized_vector(100)","2":"5200"},{"1":"vector_for_loop(100)","2":"47300"},{"1":"vectorized_vector(100)","2":"5300"},{"1":"vector_for_loop(100)","2":"51000"},{"1":"vector_for_loop(100)","2":"48100"},{"1":"vector_for_loop(100)","2":"49100"},{"1":"vector_for_loop(100)","2":"49000"},{"1":"vector_for_loop(100)","2":"50000"},{"1":"vectorized_vector(100)","2":"6300"},{"1":"vectorized_vector(100)","2":"5900"},{"1":"vectorized_vector(100)","2":"5500"},{"1":"vectorized_vector(100)","2":"5500"},{"1":"vectorized_vector(100)","2":"5000"},{"1":"vector_for_loop(100)","2":"50400"},{"1":"vectorized_vector(100)","2":"6600"},{"1":"vector_for_loop(100)","2":"41700"},{"1":"vectorized_vector(100)","2":"3600"},{"1":"vector_for_loop(100)","2":"40300"},{"1":"vector_for_loop(100)","2":"41900"},{"1":"vector_for_loop(100)","2":"41400"},{"1":"vectorized_vector(100)","2":"3700"},{"1":"vectorized_vector(100)","2":"3500"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vector_for_loop(100)","2":"42500"},{"1":"vector_for_loop(100)","2":"52600"},{"1":"vector_for_loop(100)","2":"43100"},{"1":"vector_for_loop(100)","2":"39400"},{"1":"vectorized_vector(100)","2":"3700"},{"1":"vector_for_loop(100)","2":"38600"},{"1":"vectorized_vector(100)","2":"7900"},{"1":"vectorized_vector(100)","2":"21800"},{"1":"vector_for_loop(100)","2":"38900"},{"1":"vectorized_vector(100)","2":"4300"},{"1":"vectorized_vector(100)","2":"3100"},{"1":"vector_for_loop(100)","2":"45600"},{"1":"vectorized_vector(100)","2":"6900"},{"1":"vector_for_loop(100)","2":"55000"},{"1":"vector_for_loop(100)","2":"41900"},{"1":"vector_for_loop(100)","2":"62000"},{"1":"vector_for_loop(100)","2":"49800"},{"1":"vectorized_vector(100)","2":"7500"},{"1":"vector_for_loop(100)","2":"40400"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vector_for_loop(100)","2":"41000"},{"1":"vector_for_loop(100)","2":"58000"},{"1":"vectorized_vector(100)","2":"3600"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vectorized_vector(100)","2":"3100"},{"1":"vectorized_vector(100)","2":"3000"},{"1":"vectorized_vector(100)","2":"8700"},{"1":"vector_for_loop(100)","2":"45200"},{"1":"vector_for_loop(100)","2":"47000"},{"1":"vector_for_loop(100)","2":"67000"},{"1":"vector_for_loop(100)","2":"49500"},{"1":"vector_for_loop(100)","2":"53600"},{"1":"vector_for_loop(100)","2":"50300"},{"1":"vector_for_loop(100)","2":"47600"},{"1":"vector_for_loop(100)","2":"47900"},{"1":"vectorized_vector(100)","2":"4200"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vectorized_vector(100)","2":"3400"},{"1":"vectorized_vector(100)","2":"3100"},{"1":"vector_for_loop(100)","2":"38300"},{"1":"vector_for_loop(100)","2":"37900"},{"1":"vectorized_vector(100)","2":"4100"},{"1":"vector_for_loop(100)","2":"38200"},{"1":"vector_for_loop(100)","2":"37200"},{"1":"vector_for_loop(100)","2":"37200"},{"1":"vectorized_vector(100)","2":"3300"},{"1":"vectorized_vector(100)","2":"3100"},{"1":"vectorized_vector(100)","2":"3000"},{"1":"vectorized_vector(100)","2":"3000"},{"1":"vectorized_vector(100)","2":"3000"},{"1":"vectorized_vector(100)","2":"3100"},{"1":"vectorized_vector(100)","2":"2900"},{"1":"vectorized_vector(100)","2":"3300"},{"1":"vector_for_loop(100)","2":"37700"},{"1":"vector_for_loop(100)","2":"37500"},{"1":"vector_for_loop(100)","2":"37900"},{"1":"vectorized_vector(100)","2":"3100"},{"1":"vectorized_vector(100)","2":"3000"},{"1":"vector_for_loop(100)","2":"37700"},{"1":"vector_for_loop(100)","2":"40000"},{"1":"vectorized_vector(100)","2":"3700"},{"1":"vectorized_vector(100)","2":"3300"},{"1":"vector_for_loop(100)","2":"37600"},{"1":"vectorized_vector(100)","2":"3500"},{"1":"vector_for_loop(100)","2":"41200"},{"1":"vectorized_vector(100)","2":"3500"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vector_for_loop(100)","2":"38300"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vector_for_loop(100)","2":"37700"},{"1":"vector_for_loop(100)","2":"39600"},{"1":"vectorized_vector(100)","2":"3300"},{"1":"vectorized_vector(100)","2":"3100"},{"1":"vectorized_vector(100)","2":"3700"},{"1":"vector_for_loop(100)","2":"40600"},{"1":"vector_for_loop(100)","2":"40800"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vectorized_vector(100)","2":"3100"},{"1":"vectorized_vector(100)","2":"3100"},{"1":"vector_for_loop(100)","2":"40500"},{"1":"vector_for_loop(100)","2":"39300"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vector_for_loop(100)","2":"39400"},{"1":"vector_for_loop(100)","2":"39000"},{"1":"vectorized_vector(100)","2":"3600"},{"1":"vectorized_vector(100)","2":"3100"},{"1":"vector_for_loop(100)","2":"39100"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vectorized_vector(100)","2":"3100"},{"1":"vector_for_loop(100)","2":"39100"},{"1":"vector_for_loop(100)","2":"39300"},{"1":"vector_for_loop(100)","2":"39300"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vectorized_vector(100)","2":"3300"},{"1":"vectorized_vector(100)","2":"3600"},{"1":"vectorized_vector(100)","2":"3000"},{"1":"vector_for_loop(100)","2":"39100"},{"1":"vectorized_vector(100)","2":"3300"},{"1":"vectorized_vector(100)","2":"3100"},{"1":"vectorized_vector(100)","2":"3000"},{"1":"vectorized_vector(100)","2":"3000"},{"1":"vector_for_loop(100)","2":"39300"},{"1":"vectorized_vector(100)","2":"3600"},{"1":"vector_for_loop(100)","2":"39300"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vector_for_loop(100)","2":"39100"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vectorized_vector(100)","2":"3100"},{"1":"vectorized_vector(100)","2":"3000"},{"1":"vector_for_loop(100)","2":"42400"},{"1":"vectorized_vector(100)","2":"3400"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vectorized_vector(100)","2":"3400"},{"1":"vectorized_vector(100)","2":"3200"},{"1":"vectorized_vector(100)","2":"3400"},{"1":"vector_for_loop(100)","2":"36100"},{"1":"vector_for_loop(100)","2":"36500"},{"1":"vector_for_loop(100)","2":"36800"},{"1":"vector_for_loop(100)","2":"36000"},{"1":"vectorized_vector(100)","2":"3600"},{"1":"vectorized_vector(100)","2":"3800"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

Here we use the microbenchmark function to show how many nanoseconds are needed to run each function 100 times. We can see that the vectorized function is significantly faster than its non-vectorized counterpart







