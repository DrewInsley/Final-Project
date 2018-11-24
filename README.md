---
title: "Final Project"
author: "Drew Insley"
date: "12/04/18"
output:
  html_notebook:
    theme: "readable"
---

## Set Up

```{r message=FALSE, warning=FALSE}
library(DataComputing)
library(rvest)
library(lubridate)
library(tidyr)
```

## Introduction

<span style="color:red">**The data frame used for this project comes from data.cityofnewyork.us. It is entitled "Fire Incident Dispatch Data", and it contains data spanning from 2013-2017. I intend to demonstrate my skills with R by scraping, cleaning, and using this data for analysis.**</span>

## Project

```{r echo=TRUE, message=FALSE, warning=FALSE, paged.print=FALSE}
# Get the Data
getwd()
setwd("~/Downloads")

FireIncidents <- read.csv("Fire_Incident_Dispatch_Data.csv")
```

```{r}
# Inspect the Data
str(FireIncidents)
```

```{r}
# Clean up the data
pattern <- ("[0-9]{2}[:][0-9]{2}[:][0-9]{2}")

FireIncidents <-
FireIncidents %>%
  select(INCIDENT_BOROUGH, INCIDENT_DATETIME, ENGINES_ASSIGNED_QUANTITY, INCIDENT_CLASSIFICATION_GROUP, ZIPCODE, LADDERS_ASSIGNED_QUANTITY) %>%
  rename(Borough = INCIDENT_BOROUGH, Date = INCIDENT_DATETIME, Engines = ENGINES_ASSIGNED_QUANTITY, Type = INCIDENT_CLASSIFICATION_GROUP, Ladders = LADDERS_ASSIGNED_QUANTITY, ZipCode = ZIPCODE) %>%
  mutate(Date = gsub(pattern, " ", Date)) %>%
  mutate(Date = gsub("[AMP]{2}", " ", Date)) %>%
  mutate(Date = mdy(Date))

FireIncidents %>%
  head(5)
```

<span style="color:red">**Now we have a dataset we can use. Let's make a chart of counts for incidents and ladders by borough, so that we can graph these data later.**</span>

```{r message=FALSE, warning=FALSE}
# Make a chart of counts for ladders and incidents
FireCount <-
FireIncidents %>%
  group_by(Borough) %>%
  summarise(Incidents=n()) %>%
  arrange(desc(Incidents))

FireLadders <-
FireIncidents %>%
  group_by(Borough) %>%
  summarise(Ladders=sum(Ladders)) %>%
  arrange(desc(Ladders))


FireJoin <-
  FireCount %>%
  inner_join(FireLadders)
```

<span style="color:red">**Before we check out this data, it is most important to look at specific locations where the most fires happen. We can do this by looking at fire incidents by ZipCode.**</span>

```{r}
# Where do the most fires happen?
FireIncidents %>%
  select(Borough, ZipCode) %>%
  group_by(Borough, ZipCode) %>%
  summarise(Count = n()) %>%
  arrange(desc(Count)) %>%
  filter(ZipCode != "NA") %>%
  head(5)
```

<span style="color:red">**ZipCodes 10456 and 10029 in the Bronx and Manhattan had the most incidents with 42115 and 40677 respectively. Now, let's let at which borough had the most as a whole.**</span>

```{r}
# Which borough had the most number of fire incidents?
FireJoin %>%
  ggplot(aes(reorder(Borough, desc(Incidents)), Incidents)) +
  geom_bar(stat="identity", fill="blue", color="red", alpha=.2) +
  xlab("Borough")
```

<span style="color:red">**The graph shows the descending order of fire incidents in the five boroughs of New York. Despite ZipCodes in the Bronx and Manhattan having the most fire incidents, the whole borough of Brooklyn had the most fire incidents with 777,725 occurances. Let's see if the number of ladders deployed by borough will match with the amount of incidents by borough.**</span>

```{r}
# How many ladders were deployed?
FireJoin %>%
  ggplot(aes(reorder(Borough, desc(Ladders)), Ladders)) +
  geom_bar(stat="identity", fill="red", color="blue", alpha=.2) +
  xlab("Borough")
```

<span style="color:red">**The number of ladders deployed does match with the amount of fire incidents by borough. In fact, the descending order is the exact same. To further visualize this connection, we can plot ladders and incidents on a scatterplot.**
</span>
```{r}
# Plot ladders & incidents on a scatterplot
FireJoin %>%
  ggplot(aes(Ladders, Incidents)) +
  geom_point(shape=1, aes(color= Borough)) +
  geom_smooth(method=lm, se=FALSE)
```



<span style="color:red">**As the scatterplot shows, there is a clear linear relationship between the two variables and they are heavily correlated, as expected. Let's take a look at which days these fires were most likely to occur.**</span>

```{r}
# What days were these fires most likely to occur?
FireDays <-
FireIncidents %>%
  group_by(Date) %>%
  summarise(Count=n()) %>%
  mutate(Rank = rank(desc(Count))) %>%
  select(Date, Rank) %>%
  arrange(Rank)

FireDays %>%
  head(5)
```

<span style="color:red">**Apparently people really like to start fires in February! In this case, the lower the rank, the higher the amount of fires. The 5 days with the most amounts of fires are all in the second month. However, we should look at which month had the most amount of medical emergencies, in order to determine the most dangerous month in terms of fire occurances.**</span>

```{r}
pattern <- ("[0-7]{4}[-]")

FireTypes <-
FireIncidents %>%
  select(Date, Type) %>%
  mutate(Date = gsub(pattern, " ", Date)) %>%
  mutate(Date = gsub(("[-][0-9]{2}"), " ", Date)) %>%
  group_by(Date, Type) %>%
  summarise(Emergencies=n()) %>%
  filter(Type == 'Medical Emergencies') %>%
  arrange(desc(Emergencies)) %>%
  select(Date, Emergencies)

FireTypes
```

<span style="color:red">**Oddly enough, even though the top five worst fire days were in February, the second month actually has the least amount of medical emergencies associated with those fires. In fact, the most medical emergencies come from July.**
</span>

## Conclusion

<span style="color:red">**The data shows a lot about the location and severity of fire incidents in New York from 2013-2017. The most fires occured in ZipCodes 10456 and 10029 in the Bronx and Manhattan, but Brooklyn had the most fires as a whole, as well as the most fire ladders deployed. In addition, the top five worst days with respect to amount of fires were all in February, but the month with the most medical emergencies related to these fires was actually July.**
</span>

