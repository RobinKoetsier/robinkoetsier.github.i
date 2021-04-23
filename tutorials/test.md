---
layout: post
title:  "Percentile Radars"
date:   2021-22-04 10:00:40
blurb: "Making radars"
og_image: https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/second_post/0.jpg
---

As I really like the radars from football slices (RIP) and the mplsoccer package for Python, I was thinking about making them in R with the help of the [worldfootballR package.](https://github.com/JaseZiv/worldfootballR). When some people contacted me with the question if I knew how to do it, I decided to make a tutorial for it. If you don't know what I'm talking about, this is the radar from mplsoccer. 
<p align="center">
<img width="400" alt="Pizza Plot" src="https://mplsoccer.readthedocs.io/en/latest/_images/sphx_glr_plot_pizza_colorful_001.png">
</p>
The Python package makes you enter te values yourself. For some leagues (Men's Big 5 Leagues and European Competition, Major League Soccer, Women's Super League) FBref has so called scouting reports with data from StatsBomb. These scouting reports do not only have the absolute numers for several metrics, but also the percentiles. 
The worldfootballR package let's you scrape theme really easy. Let's start by setting up out environment.

```{r}
library(worldfootballR)  #for scraping
library(tidyverse)    #for ggplot, dplyr and several other stuff
library(ggtext) #for text manipulation
```
