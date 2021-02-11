---
title: 'Weekly Exercises #3'
author: "Elizabeth Cain"
output: 
  html_document:
    keep_md: TRUE
    toc: TRUE
    toc_float: TRUE
    df_print: paged
    code_download: true
---
```{r setup, include=FALSE}
#knitr::opts_chunk$set(echo = TRUE, error=TRUE, message=FALSE, warning=FALSE)
```

```{r libraries}
library(tidyverse)     # for graphing and data cleaning
library(gardenR)       # for Lisa's garden data
library(lubridate)     # for date manipulation
library(ggthemes)      # for even more plotting themes
library(geofacet)      # for special faceting with US map layout
theme_set(theme_minimal())       # My favorite ggplot() theme :)
```

```{r data}
# Lisa's garden data
data("garden_harvest")

# Seeds/plants (and other garden supply) costs
data("garden_spending")

# Planting dates and locations
data("garden_planting")

# Tidy Tuesday data
kids <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-09-15/kids.csv')
```

## Setting up on GitHub!

Before starting your assignment, you need to get yourself set up on GitHub and make sure GitHub is connected to R Studio. To do that, you should read the instruction (through the "Cloning a repo" section) and watch the video [here](https://github.com/llendway/github_for_collaboration/blob/master/github_for_collaboration.md). Then, do the following (if you get stuck on a step, don't worry, I will help! You can always get started on the homework and we can figure out the GitHub piece later):

* Create a repository on GitHub, giving it a nice name so you know it is for the 3rd weekly exercise assignment (follow the instructions in the document/video).  
* Copy the repo name so you can clone it to your computer. In R Studio, go to file --> New project --> Version control --> Git and follow the instructions from the document/video.  
* Download the code from this document and save it in the repository folder/project on your computer.  
* In R Studio, you should then see the .Rmd file in the upper right corner in the Git tab (along with the .Rproj file and probably .gitignore).  
* Check all the boxes of the files in the Git tab and choose commit.  
* In the commit window, write a commit message, something like "Initial upload" would be appropriate, and commit the files.  
* Either click the green up arrow in the commit window or close the commit window and click the green up arrow in the Git tab to push your changes to GitHub.  
* Refresh your GitHub page (online) and make sure the new documents have been pushed out.  
* Back in R Studio, knit the .Rmd file. When you do that, you should have two (as long as you didn't make any changes to the .Rmd file, in which case you might have three) files show up in the Git tab - an .html file and an .md file. The .md file is something we haven't seen before and is here because I included `keep_md: TRUE` in the YAML heading. The .md file is a markdown (NOT R Markdown) file that is an interim step to creating the html file. They are displayed fairly nicely in GitHub, so we want to keep it and look at it there. Click the boxes next to these two files, commit changes (remember to include a commit message), and push them (green up arrow).  
* As you work through your homework, save and commit often, push changes occasionally (maybe after you feel finished with an exercise?), and go check to see what the .md file looks like on GitHub.  
* If you have issues, let me know! This is new to many of you and may not be intuitive at first. But, I promise, you'll get the hang of it!

## Instructions

* Put your name at the top of the document. 

* **For ALL graphs, you should include appropriate labels.** 

* Feel free to change the default theme, which I currently have set to `theme_minimal()`. 

* Use good coding practice. Read the short sections on good code with [pipes](https://style.tidyverse.org/pipes.html) and [ggplot2](https://style.tidyverse.org/ggplot2.html). **This is part of your grade!**

* When you are finished with ALL the exercises, uncomment the options at the top so your document looks nicer. Don't do it before then, or else you might miss some important warnings and messages.

## Warm-up exercises with garden data

These exercises will reiterate what you learned in the "Expanding the data wrangling toolkit" tutorial. If you haven't gone through the tutorial yet, you should do that first.

  1. Summarize the `garden_harvest` data to find the total harvest weight in pounds for each vegetable and day of week (HINT: use the `wday()` function from `lubridate`). Display the results so that the vegetables are rows but the days of the week are columns.

```{r}
garden_harvest %>%
  mutate(day = wday(date, label = TRUE)) %>% 
  group_by(vegetable, day) %>% 
  mutate(weight_lbs=weight*0.00220462) %>% 
  summarize(total_weight_lbs = sum(weight_lbs)) %>% 
  pivot_wider(id_cols = vegetable,
              names_from = day,
              values_from = total_weight_lbs)
```

  2. Summarize the `garden_harvest` data to find the total harvest in pound for each vegetable variety and then try adding the plot from the `garden_planting` table. This will not turn out perfectly. What is the problem? How might you fix it?

```{r}
garden_harvest %>%
  group_by(vegetable, variety) %>% 
  summarize(total_weight_lbs = sum((weight)*0.00220462)) %>%
  left_join(garden_planting,
            by = c("vegetable","variety"))
```

The problem is that some of the plot data is unknown and some filled in by the many options such that we could have some repetition of cases with differing plot options.

3. I would like to understand how much money I "saved" by gardening, for each vegetable type. Describe how I could use the `garden_harvest` and `garden_spending` data sets, along with data from somewhere like [this](https://products.wholefoodsmarket.com/search?sort=relevance&store=10542) to answer this question. You can answer this in words, referencing various join functions. You don't need R code but could provide some if it's helpful.

```{r}
garden_harvest %>%
  group_by(vegetable, variety) %>% 
  summarize(total_weight_lbs = sum((weight)*0.00220462)) %>%
  left_join(garden_spending,
            by = c("vegetable","variety"))
```

We could figure out how much money you "saved" by gardening by summarizing the `garden_harvest` data to find the total harvest in pound for each vegetable variety joined by the `garden_spending` information on price of each variety of vegetable. From there, we could use the market prices for vegetables ($/pound) along with the total weight of the harvested vegetables to see what the cost at the supermarket would have been (simple multiplication). We can compare that to the spending information to find the net savings with simple subtraction.

4. Subset the data to tomatoes. Reorder the tomato varieties from smallest to largest first harvest date. Create a bar plot of total harvest in pounds for each variety, in the new order.

```{r}
garden_harvest %>% 
  filter(vegetable == "tomatoes") %>%
  mutate(weight_lbs=weight*0.00220462) %>% 
  arrange(date) %>% 
  ggplot(aes(y=weight_lbs, x=date))+
  geom_col()+
  facet_wrap(~fct_reorder(factor(variety), date))+
  #facet_wrap(vars(variety))+
  labs(title = "Total Harvest in Pounds",
       x="Month",
       y="Pounds")
```

  5. In the `garden_harvest` data, create two new variables: one that makes the varieties lowercase and another that finds the length of the variety name. Arrange the data by vegetable and length of variety name (smallest to largest), with one row for each vegetable variety. HINT: use `str_to_lower()`, `str_length()`, and `distinct()`.
  
```{r}
garden_harvest %>% 
  select(vegetable,variety) %>% 
  mutate(lowercase= str_to_lower(variety),
         length=str_length(variety)) %>% 
  #arrange(length) %>% 
  arrange(vegetable,length) %>% 
  distinct()
```

  6. In the `garden_harvest` data, find all distinct vegetable varieties that have "er" or "ar" in their name. HINT: `str_detect()` with an "or" statement (use the | for "or") and `distinct()`.

```{r}
garden_harvest %>% 
  mutate(er_ar=str_detect(variety,"er|ar")) %>% 
  #filter(er_ar==TRUE)
  distinct()
```


## Bicycle-Use Patterns

In this activity, you'll examine some factors that may influence the use of bicycles in a bike-renting program.  The data come from Washington, DC and cover the last quarter of 2014.

Two data tables are available:

- `Trips` contains records of individual rentals
- `Stations` gives the locations of the bike rental stations

Here is the code to read in the data. We do this a little differently than usually, which is why it is included here rather than at the top of this file. To avoid repeatedly re-reading the files, start the data import chunk with `{r cache = TRUE}` rather than the usual `{r}`.

```{r cache=TRUE}
data_site <- 
  "https://www.macalester.edu/~dshuman1/data/112/2014-Q4-Trips-History-Data.rds" 
Trips <- readRDS(gzcon(url(data_site)))
Stations<-read_csv("http://www.macalester.edu/~dshuman1/data/112/DC-Stations.csv")
```

**NOTE:** The `Trips` data table is a random subset of 10,000 trips from the full quarterly data. Start with this small data table to develop your analysis commands. **When you have this working well, you should access the full data set of more than 600,000 events by removing `-Small` from the name of the `data_site`.**

### Temporal patterns

It's natural to expect that bikes are rented more at some times of day, some days of the week, some months of the year than others. The variable `sdate` gives the time (including the date) that the rental started. Make the following plots and interpret them:

  7. A density plot, which is a smoothed out histogram, of the events versus `sdate`. Use `geom_density()`.
  
```{r}
Trips %>% 
  ggplot(aes(x=sdate))+
  geom_density()+
  theme(axis.text.y = element_blank())+
  labs(title="Rental Density as a Function of Time", x="Date", y="Density")
```
  
From this density plot, we can see that generally, as the time progressed from October to January, the rental popularity decreased. This makes sense when we think about the weather getting colder. Likewise, some of the spikes can likely be explained by some nice weather days. We do see a significant dip at some point in December, I think this might be explained by people traveling for the holidays. I think other peaks could also be explain by seeing if there were important events at that date (ex. Election day in Nov).
  
  8. A density plot of the events versus time of day.  You can use `mutate()` with `lubridate`'s  `hour()` and `minute()` functions to extract the hour of the day and minute within the hour from `sdate`. Hint: A minute is 1/60 of an hour, so create a variable where 3:30 is 3.5 and 3:45 is 3.75.
  
```{r}
Trips %>% 
  mutate(hour=hour(sdate),
         min=(minute(sdate))/60,
         time=hour+min) %>% 
  ggplot(aes(x=time))+
  geom_density()+
  theme(axis.text.y = element_blank())+
  labs(title="Rental Density as a Function of Time", x="Time", y="Density")
```

From this density plot we can see that the typical morning commute time and the typical end of workday commute times are the most popular. Around lunch time also seems to be pretty popular. Late night/early morning are not popular rental start times.

  9. A bar graph of the events versus day of the week. Put day on the y-axis.
  
```{r}
Trips %>% 
  mutate(day=wday(sdate,label=TRUE)) %>% 
  ggplot(aes(y=day))+
  geom_bar()+
  labs(title = "Rental Events by Day",
       x="Rentals",
       y="")
```

From this bar graph, we can see that Friday, Thursday, and Monday are fairly popular rental days. Sunday is the least popular day for rentals.

  10. Facet your graph from exercise 8. by day of the week. Is there a pattern?
  
```{r}
Trips %>% 
  mutate(hour=hour(sdate),
         min=(minute(sdate))/60,
         time=hour+min,
         day=wday(sdate,label=TRUE)) %>% 
  ggplot(aes(x=time))+
  geom_density()+
  facet_wrap(~day)+
  theme(axis.text.y = element_blank())+
  labs(title="Rental Density as a Function of Time", x="Time", y="Density")
```

From these plots, we can see that the popular times of day are not the same on weekdays and weekends. The weekdays have the typical commute peaks, and the weekends show peaks in the mid-morning through afternoon times. The pattern is very well defined.

The variable `client` describes whether the renter is a regular user (level `Registered`) or has not joined the bike-rental organization (`Causal`). The next set of exercises investigate whether these two different categories of users show different rental behavior and how `client` interacts with the patterns you found in the previous exercises. 

  11. Change the graph from exercise 10 to set the `fill` aesthetic for `geom_density()` to the `client` variable. You should also set `alpha = .5` for transparency and `color=NA` to suppress the outline of the density function.
  
```{r}
Trips %>% 
  mutate(hour=hour(sdate),
         min=(minute(sdate))/60,
         time=hour+min,
         day=wday(sdate,label=TRUE)) %>% 
  ggplot(aes(x=time, fill=client))+
  geom_density(alpha=0.5, color=NA)+
  facet_wrap(~day)+
  theme_base()+
  theme(axis.text.y = element_blank())+
  labs(title="Rental Density as a Function of Time", x="Time", y="Density", fill="Client Type")
```

From these plots, it seems that the commute peaks during the weekdays come from registered users mainly. Casual users appear to typically initiate their rentals mid-morning through afternoon hours regardless of the day of the week. On the weekends, both registered and casual users have similar peak times. Also from these graphs, we can see that more casual clients tend to rent the bikes on the weekend.

  12. Change the previous graph by adding the argument `position = position_stack()` to `geom_density()`. In your opinion, is this better or worse in terms of telling a story? What are the advantages/disadvantages of each?
  
```{r}
Trips %>% 
  mutate(hour=hour(sdate),
         min=(minute(sdate))/60,
         time=hour+min,
         day=wday(sdate,label=TRUE)) %>% 
  ggplot(aes(x=time, fill=client))+
  geom_density(alpha=0.5, color=NA, position = position_stack())+
  facet_wrap(~day)+
  theme_base()+
  theme(axis.text.y = element_blank())+
  labs(title="Rental Density as a Function of Time", x="Time", y="Density", fill="Client Type")
```

In my opinion, is this worse in terms of telling a story because the density comparison between the two groups is not as clear. However, this does give a better idea of the overall density. Also, this approach does not give as narrow time scopes for the peaks, which can impact the interpretation of the information. Also, it is harder to see that difference in pattern between the casual and registered groups.

  13. In this graph, go back to using the regular density plot (without `position = position_stack()`). Add a new variable to the dataset called `weekend` which will be "weekend" if the day is Saturday or Sunday and  "weekday" otherwise (HINT: use the `ifelse()` function and the `wday()` function from `lubridate`). Then, update the graph from the previous problem by faceting on the new `weekend` variable. 
  
```{r}
Trips %>% 
  mutate(hour=hour(sdate),
         min=(minute(sdate))/60,
         time=hour+min,
         day=wday(sdate,label=TRUE),
         weekend = ifelse(day %in% c("Sat","Sun"), "Weekend", "Weekday")) %>% 
  ggplot(aes(x=time, fill=client))+
  geom_density(alpha=0.5, color=NA)+
  facet_wrap(~weekend)+
  theme_clean()+
  theme(axis.text.y = element_blank())+
  labs(title="Rental Density as a Function of Time", x="Time", y="Density", fill="Client Type")
```

From this graphic, we learn that weekend rental popularity is higher than weekday rental popularity for casual users but that registered users account for the most rentals in their peak times during the weekdays. This graphic is also really nice because we can still see the main takeaways of commute times on weekdays and mid-morning through afternoon on weekends being peaks for registered users and mid-morning through afternoon being peak times for casual users regardless of day.
  
  14. Change the graph from the previous problem to facet on `client` and fill with `weekday`. What information does this graph tell you that the previous didn't? Is one graph better than the other?
  
```{r}
Trips %>% 
  mutate(hour=hour(sdate),
         min=(minute(sdate))/60,
         time=hour+min,
         day=wday(sdate,label=TRUE),
         weekend = ifelse(day %in% c("Sat","Sun"), "Weekend", "Weekday")) %>% 
  ggplot(aes(x=time, fill=weekend))+
  geom_density(alpha=0.5, color=NA)+
  facet_wrap(~client)+
  theme_clean()+
  theme(axis.text.y = element_blank())+
  labs(title="Rental Density as a Function of Time", x="Time", y="Density", fill="Key")
```

This graph tells makes it easier to see the difference between the registered users' habits on weekdays and weekends. I think the previous graph is better than this one because it better highlights the patterns present in the data by focusing the attention on the difference in time (something about splitting the graph into two time categories works better with time as the x variable in my opinion).
    
### Spatial patterns

  15. Use the latitude and longitude variables in `Stations` to make a visualization of the total number of departures from each station in the `Trips` data. Use either color or size to show the variation in number of departures. We will improve this plot next week when we learn about maps!
  
```{r}
Trips %>% 
  left_join(Stations,by=c("sstation"="name")) %>% 
  group_by(lat, long) %>% 
  summarise(departures=n()) %>% 
  ggplot()+
  geom_point(aes(y=lat,x=long, color=departures))+
  labs(title="Spacial Departure Popularity", x="Longitude", y="Latitude")
```
  
  16. Only 14.4% of the trips in our data are carried out by casual users. Create a plot that shows which area(s) have stations with a much higher percentage of departures by casual users. What patterns do you notice? (Again, we'll improve this next week when we learn about maps).
  
```{r}
Trips %>% 
  left_join(Stations,by=c("sstation"="name")) %>% 
  group_by(lat, long) %>%
  summarise(prop=sum(client=="Casual")/n()) %>% 
  ggplot()+
  geom_point(aes(y=lat,x=long, color=prop))+
  labs(title="Spacial Departure Popularity by Casual User Status", x="Longitude", y="Latitude", color="Proportion of Casual Users")
```

We can see that locations with more rentals by casual users are located in what seems to be a dowtown/popular area. However, I do not notice any clear patterns
  
### Spatiotemporal patterns

  17. Make a table with the ten station-date combinations (e.g., 14th & V St., 2014-10-14) with the highest number of departures, sorted from most departures to fewest. Save this to a new dataset and print out the dataset. Hint: `as_date(sdate)` converts `sdate` from date-time format to date format. 
  
```{r}
TopTen <- 
  Trips %>% 
  mutate(plain_date=as_date(sdate)) %>%
  group_by(sstation,plain_date) %>% 
  summarise(departures=n()) %>% 
  arrange(desc(departures)) %>% 
  head(n=10)

TopTen
```
  
  18. Use a join operation to make a table with only those trips whose departures match those top ten station-date combinations from the previous part.
  
```{r}
Trips %>% 
  mutate(plain_date=as_date(sdate)) %>%
  inner_join(TopTen, by=c("sstation","plain_date"))
```
  
  19. Build on the code from the previous problem (ie. copy that code below and then %>% into the next step.) and group the trips by client type and day of the week (use the name, not the number). Find the proportion of trips by day within each client type (ie. the proportions for all 7 days within each client type add up to 1). Display your results so day of week is a column and there is a column for each client type. Interpret your results.

```{r}
Trips %>% 
  mutate(plain_date=as_date(sdate)) %>%
  inner_join(TopTen, by=c("sstation","plain_date")) %>% 
  mutate(weekday=(wday(sdate, label=TRUE))) %>% 
  group_by(client, weekday) %>%
  summarise(client_depart=n()) %>% 
  group_by(client) %>% 
  mutate(prop =client_depart/sum(client_depart)) %>% 
  pivot_wider(id_cols=weekday,names_from=client, values_from=prop)
```

On the weekends, more casual users are renting bikes than registered users. On the weekdays, more registered users are renting the bikes than casual users. In both situations, the gap between casual and registered users is quite considerable. It seems that Thursday is by far the most popular day for registered users and Saturday for casual.

**DID YOU REMEMBER TO GO BACK AND CHANGE THIS SET OF EXERCISES TO THE LARGER DATASET? IF NOT, DO THAT NOW.**

## GitHub link

  20. Below, provide a link to your GitHub page with this set of Weekly Exercises. Specifically, if the name of the file is 03_exercises.Rmd, provide a link to the 03_exercises.md file, which is the one that will be most readable on GitHub.

[link](https://github.com/ecain21/Assignment-3/blob/main/03_exercises.md)

## Challenge problem! 

This problem uses the data from the Tidy Tuesday competition this week, `kids`. If you need to refresh your memory on the data, read about it [here](https://github.com/rfordatascience/tidytuesday/blob/master/data/2020/2020-09-15/readme.md). 

  21. In this exercise, you are going to try to replicate the graph below, created by Georgios Karamanis. I'm sure you can find the exact code on GitHub somewhere, but **DON'T DO THAT!** You will only be graded for putting an effort into this problem. So, give it a try and see how far you can get without doing too much googling. HINT: use `facet_geo()`. The graphic won't load below since it came from a location on my computer. So, you'll have to reference the original html on the moodle page to see it.
  
```{r, eval=FALSE}
kids %>%
  filter(variable=="lib") %>%
  ? pivot_wider(id_cols=state, #pull years out so we can take the difference but we still want years in it for graphing x=year?)
  ? mutate(short = year_2016 - year_1997) %>% 
  ggplot()+
  facet_geo(vars(state), scales="free") +
  geom_line(aes(x=year, y=diff))
```

**DID YOU REMEMBER TO UNCOMMENT THE OPTIONS AT THE TOP?**
