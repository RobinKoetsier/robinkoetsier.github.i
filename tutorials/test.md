---
layout: post
title:  "Percentile Radars"
date:   24-04-2021
blurb: "Making radars"
og_image: https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/second_post/0.jpg
---

As I really like the radars from football slices (RIP) and the mplsoccer package for Python, I was thinking about making them in R with the help of the [worldfootballR package.](https://github.com/JaseZiv/worldfootballR). When some people contacted me with the question if I knew how to do it, I decided to make a tutorial for it. If you don't know what I'm talking about, this is the radar from mplsoccer. 
<p align="center">
<img width="400" alt="Pizza Plot" src="https://mplsoccer.readthedocs.io/en/latest/_images/sphx_glr_plot_pizza_colorful_001.png">
</p>
The Python package makes you enter te values yourself. For some leagues (Men's Big 5 Leagues and European Competition, Major League Soccer, Women's Super League) FBref has so called scouting reports with data from StatsBomb. These scouting reports do not only have the absolute numbers for several metrics, but also the percentiles. 
The worldfootballR package let's you scrape them really easy. Let's start by setting up out environment. If needed, install the packages first.

  
```r
library(worldfootballR)  #for scraping
library(tidyverse)       #for ggplot, dplyr and several other stuff
library(ggtext)          #for text manipulation
```

Next we are going to pick a player in which we are interested, as long as it's from the leagues mentioned earlier. I'm choosing Mateusz Klich, but you can pick someone else, I'm not judging you.
As mentioned before, worldfootballR has a function to scrape the scouting report.

```r
df <- fb_player_scouting_report(https://fbref.com/en/players/282679b4/Mateusz-Klich)
head(df_klich)
```
To colour them by type of the Statistic, we make a new column and fill it with "Attacking", "Possession" or "Defending"

```r
df <- df %>% mutate(stat=case_when(Statistic == "Non-Penalty Goals"|
                                               Statistic == "npxG"|
                                               Statistic == "Shots Total"|
                                               Statistic == "Assists"|
                                               Statistic == "xA"|
                                               Statistic == "npxG+xA"|
                                               Statistic == "Shot-Creating Actions" ~ "Attacking",
                                               Statistic == "Passes Attempted"|
                                               Statistic == "Pass Completion %"|
                                               Statistic == "Progressive Passes"|
                                               Statistic == "Progressive Carries"|
                                               Statistic == "Dribbles Completed"|
                                               Statistic == "Touches (Att Pen)"|
                                               Statistic == "Progressive Passes Rec" ~ "Possession",
                                             TRUE ~ "Defending"))
```



```
frenkie <- frenkie %>% mutate(stat=case_when(Statistic == "Non-Penalty Goals"|
                                               Statistic == "npxG"|
                                               Statistic == "Shots Total"|
                                               Statistic == "Assists"|
                                               Statistic == "xA"|
                                               Statistic == "npxG+xA"|
                                               Statistic == "Shot-Creating Actions" ~ "Attacking",
                                               Statistic == "Passes Attempted"|
                                               Statistic == "Pass Completion %"|
                                               Statistic == "Progressive Passes"|
                                               Statistic == "Progressive Carries"|
                                               Statistic == "Dribbles Completed"|
                                               Statistic == "Touches (Att Pen)"|
                                               Statistic == "Progressive Passes Rec" ~ "Possession",
                                             TRUE ~ "Defending"))
```
