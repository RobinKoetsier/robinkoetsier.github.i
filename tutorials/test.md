---
layout: post
title:  "Percentile Radars"
date:   24-04-2021
blurb: "Making radars"
og_image: https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/second_post/0.jpg
---

As I really like the radars from football slices (RIP) and the mplsoccer package for Python, I was thinking about making them in R with the help of the [worldfootballR package.](https://github.com/JaseZiv/worldfootballR). When some people contacted me with the question if I knew how to do it, I decided to make a tutorial for it. If you don't know what I'm talking about, this is the radar from mplsoccer. 

<p align="center">
  
  <a href="https://mplsoccer.readthedocs.io/en/latest/_images/sphx_glr_plot_pizza_colorful_001.png">
<img src="https://mplsoccer.readthedocs.io/en/latest/_images/sphx_glr_plot_pizza_colorful_001.png"
     style="width:400px">
  </a>
</p>

The Python package makes you enter te values yourself. For some leagues (Men's Big 5 Leagues and European Competition, Major League Soccer, Women's Super League) FBref has so called scouting reports with data from StatsBomb. These scouting reports do not only have the absolute numbers for several metrics, but also the percentiles. 
The worldfootballR package let's you scrape them really easy. Let's start by setting up our environment. If needed, install the packages first.

  
```r
library(worldfootballR)  #for scraping
library(tidyverse)       #for ggplot, dplyr and several other stuff
library(ggtext)          #for text manipulation
library(glue)            #easier than paste()
```

Next we are going to pick a player in which we are interested, as long as it's from the leagues mentioned earlier. I'm choosing Mateusz Klich, but you can pick someone else, I'm not judging you.
As mentioned before, worldfootballR has a function to scrape the scouting report.

```
df <- fb_player_scouting_report(https://fbref.com/en/players/282679b4/Mateusz-Klich)
```
To colour them by type of the Statistic, we make a new column and fill it with "Attacking", "Possession" or "Defending". 

```r
df <- df %>% 
      mutate(stat=case_when(Statistic == "Non-Penalty Goals"|
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

Now we can use this column to color the chart. I'm not interested in every metric though. The 'npxG+xA' column for example. I already have those metrics in my chart. To pick the metrics you want/don't want, print the Statistic column and choose.

```r
print(df$Statistic)
 [1] "Non-Penalty Goals"      "npxG"                   "Shots Total"            "Assists"               
 [5] "xA"                     "npxG+xA"                "Shot-Creating Actions"  "Passes Attempted"      
 [9] "Pass Completion %"      "Progressive Passes"     "Progressive Carries"    "Dribbles Completed"    
[13] "Touches (Att Pen)"      "Progressive Passes Rec" "Pressures"              "Tackles"               
[17] "Interceptions"          "Blocks"                 "Clearances"             "Aerials won"        
```

I decided to exclude some instead of picking the one to include. This was purely because is was less code.

```r
df_selected <- df[-c(6,14,15,16,18),]
```

If you want to pick your metric, use the same statement without the minus sign. So:

```r
df_selected <- df[c(1:5,7:13,17,19,20),]
```

You will get the same result.

To make the pizza chart, we will use geom_bar() with coord_polar(). It's a neat little trick as there isn't a good package to do it otherwise. 

```r
ggplot(df_selected,aes(fct_reorder(Statistic,stat),Percentile)) +                       #select the columns to plot and sort it so the types of metric are grouped
  geom_bar(aes(y=100,fill=stat),stat="identity",width=1,colour="white",                 #make the whole pizza first
  alpha=0.5) +                                                                          #change alphe to make it more or less visible
  geom_bar(stat="identity",width=1,aes(fill=stat),colour="white") +                     #insert the values 
  coord_polar() +                                                                       #make it round
  geom_label(aes(label=Per.90,fill=stat),size=2,color="white",show.legend = FALSE)+     #add a label for the value. Change 'label=Per.90' to 'label=Percentile' to show the percentiles
 scale_fill_manual(values=c("Possession" = "#D70232",                                   #choose colors to fill the pizza parts
                             "Attacking" = "#1A78CF",
                             "Defending" = "#FF9300")) +                                                              
  scale_y_continuous(limits = c(-10,100))+                                              #create the white part in the middle.   
  labs(fill="",                                                                         #remove legend title
       caption = "Data from StatsBomb via FBref",                                        #credit FBref/StatsBomb
       title=df_selected$player_name[1])+                                               #let the title be te name of the player
 
  theme_minimal() +                                                                     #from here it's only themeing. 
  theme(legend.position = "top",
        axis.title.y = element_blank(),
        axis.title.x = element_blank(),
        axis.text.y = element_blank(),
        text = element_text(family="Spartan-Light"),                                    #I downloaded this font from Google Fonts. You can use your own font of course
        plot.title = element_text(hjust=0.5),
        plot.caption = element_text(hjust=0.5,size=6),
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank()) 
```
<p align="center">
   <a href="https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/tutorials/pizza/first-1.png">
<img src="https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/tutorials/pizza/first-1.png" style="width:400px">
      </a>
</p>
Doesn't look half bad, but the labels are horrible. Besides that we should add some extra information about the player. The labels need to be rotated so they look better. We can do this by hand, but if we change the number of metrics we're using we need to do it all over again. So let's just make a calculation that we can run everytime we make a new chart.

```r
temp <- (360/(length(df_selected$player_name))/2)                             #find the difference in angle between to labels and divide by two.
myAng <- seq(-temp, -360+temp, length.out = length(df_selected$player_name))  #get the angle for every label
ang<-ifelse(myAng < -90, myAng+180, myAng)                                    #rotate label by 180 in some places for readability
ang<-ifelse(ang < -90, ang+180, ang)                                          #rotate some lables back for readability...
```
Because some labels are rather long ('Progressive Passes Rec' for instance) I decided to let every word start on a new line. I used gsub for that

```r
df_selected$Statistic <- gsub(" ","\n",df_selected$Statistic)
```
If we plot again and add an extra line we will get a better plot.

```r
ggplot(df_selected,aes(fct_reorder(Statistic,stat),Percentile)) +                       #select the columns to plot and sort it so the types of metric are grouped
  geom_bar(aes(y=100,fill=stat),stat="identity",width=1,colour="white",                 #make the whole pizza first
  alpha=0.5) +                                                                          #change alphe to make it more or less visible
  geom_bar(stat="identity",width=1,aes(fill=stat),colour="white") +                     #insert the values 
  coord_polar() +                                                                       #make it round
  geom_label(aes(label=Per.90,fill=stat),size=2,color="white",show.legend = FALSE)+     #add a label for the value. Change 'label=Per.90' to 'label=Percentile' to show the percentiles
 scale_fill_manual(values=c("Possession" = "#D70232",                                   #choose colors to fill the pizza parts
                             "Attacking" = "#1A78CF",
                             "Defending" = "#FF9300")) +                                                              
  scale_y_continuous(limits = c(-10,100))+                                              #create the white part in the middle.   
  labs(fill="",                                                                         #remove legend title
       caption = "Data from StatsBomb via FBref",                                        #credit FBref/StatsBomb
       title=df_selected$player_name[1])+                                               #let the title be te name of the player
 
  theme_minimal() +                                                                     #from here it's only themeing. 
  theme(legend.position = "top",
        axis.title.y = element_blank(),
        axis.title.x = element_blank(),
        axis.text.y = element_blank(),
        axis.text.x = element_text(size = 6, angle = ang),
        text = element_text(family="Spartan-Light"),                                    #I downloaded this font from Google Fonts. You can use your own font of course
        plot.title = element_text(hjust=0.5),
        plot.caption = element_text(hjust=0.5,size=6),
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank()) 
```
<p align="center">
   <a href="https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/tutorials/pizza/second-1.png">
<img src="https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/tutorials/pizza/second-1.png" style="width:400px">
      </a>
</p>

That looks much better! From here you can change everything you want. I'm going to add a subtitle, and make some theme adjustments. Adding a picture is something I will add to this tutorial in the future.

<p align="center">
   <a href="https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/tutorials/pizza/third-2.png">
<img src="https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/tutorials/pizza/third-2.png" style="width:400px">
      </a>
</p>

{::options parse_block_html="true" /}

<details><summary markdown="span">Let's see de final code!</summary>
  
```r
ggplot(df_selected,aes(fct_reorder(Statistic,stat),Percentile)) +                       #select the columns to plot and sort it so the types of metric are grouped
  geom_bar(aes(y=100,fill=stat),stat="identity",width=1,colour="white",                 #make the whole pizza first
  alpha=0.5) +                                                                          #change alphe to make it more or less visible
  geom_bar(stat="identity",width=1,aes(fill=stat),colour="white") +                     #insert the values 
  coord_polar() +                                                                       #make it round
  geom_label(aes(label=Per.90,fill=stat),size=2,color="white",show.legend = FALSE)+     #add a label for the value. Change 'label=Per.90' to 'label=Percentile' to show the percentiles
 scale_fill_manual(values=c("Possession" = "#D70232",                                   #choose colors to fill the pizza parts
                             "Attacking" = "#1A78CF",
                             "Defending" = "#FF9300")) +                                                              
  scale_y_continuous(limits = c(-10,100))+                                              #create the white part in the middle.   
  labs(fill="",   
       caption = "Data from StatsBomb via FBref",     
       #remove legend title
       title=glue("{df_selected$player_name[1]} | Leeds United"),
        subtitle = glue::glue("{df_selected$season} | Compared to midfielders Top 5 competitions | stats per 90"))+ #let the title be te name of the player                                                
 
  theme_minimal() +                                                                     #from here it's only themeing. 
  theme(plot.background = element_rect(fill = "#F2F4F5",color = "#F2F4F5"),
        panel.background = element_rect(fill = "#F2F4F5",color = "#F2F4F5"),
        legend.position = "top",
        axis.title.y = element_blank(),
        axis.title.x = element_blank(),
        axis.text.y = element_blank(),
         axis.text.x = element_text(size = 6, angle = ang),
        text = element_text(family="Spartan-Light"),                                    #I downloaded this font from Google Fonts. You can use your own font of course
        plot.title = element_markdown(hjust=0.5,family="Spartan-Medium"),
        plot.subtitle = element_text(hjust=0.5,size=8),
        plot.caption = element_text(hjust=0.5,size=6),
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank(),
        plot.margin = margin(5,2,2,2)) 
```

Not too hard to do, right?
</details>
<br/>

{::options parse_block_html="false" /}

Note: with coord_polar() in combination with a background colour, I need to trim the image afterwards. I do this with a commandline (on mac OSX) statement in my R session:

```r
system("convert -trim image.png new_image.png")
```
Where image.png is my just saved image and new_image.png will be the trimmed one. 
There is also an option to add the color to ggsave:

```r
ggsave("image.png",bg="#F2F4F5")
```

I prefer to trim it, as it removes unnecessary parts from the plot.

#### Some more styles

The Athletic/ Tom Worville
<p align="center">
   <a href="https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/tutorials/pizza/athe1.png">
<img src="https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/tutorials/pizza/athe1.png" style="width:400px">
      </a>
</p>


{::options parse_block_html="true" /}

<details><summary markdown="span">The code</summary>
  
```r
ggplot(df_selected,aes(fct_reorder(Statistic,stat),Percentile)) +                       
  geom_bar(aes(y=100),fill="#131313",stat="identity",width=1,colour="#797979",                 
  alpha=0.5,show.legend = FALSE) +      
  
  
  geom_bar(stat="identity",width=1,aes(fill=stat),colour="#F3FEFC",alpha=1) +                     
  coord_polar(clip = "off") +                                                                      
     geom_hline(yintercept=25, colour="#565656",linetype="longdash",alpha=0.5)+
  geom_hline(yintercept=50, colour="#565656",linetype="longdash",alpha=0.5)+
  geom_hline(yintercept=75, colour="#565656",linetype="longdash",alpha=0.5)+ 
 scale_fill_manual(values=c("Possession" = "#1ADA89",                                   
                             "Attacking" = "#0F70BF",
                             "Defending" = "#EC313A")) +                                                        
   geom_label(aes(label=Percentile,fill=stat),size=2,color="white",show.legend = FALSE)+ 
  scale_y_continuous(limits = c(-20,100))+                                              
  labs(fill="",   
       caption = "Data from StatsBomb via FBref\nStyle copied from The Athletic/@worville",     
       #remove legend title
       title=glue("{df_selected$player_name[1]} | Leeds United"),
        subtitle = glue::glue("{df_selected$season} | Compared to midfielders Top 5 competitions | stats per 90"))+                                                
  theme_minimal() +                                                                     
  theme(plot.background = element_rect(fill = "#131313",color = "#131313"),
        panel.background = element_rect(fill = "#131313",color = "#131313"),
        legend.position = "bottom",
        axis.title.y = element_blank(),
        axis.title.x = element_blank(),
        axis.text.y = element_blank(),
        axis.text.x = element_text(size = 6,colour = "#FFFFFF"),
        text = element_text(family="Spartan-Light",colour= "#FEFEFE"),                                   
        plot.title = element_markdown(hjust=0.5,family="Spartan-Medium"),
        plot.subtitle = element_text(hjust=0.5,size=8),
        plot.caption = element_text(hjust=0.5,size=6),
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank(),
        plot.margin = margin(5,4,2,4)) 
```


</details>
<br/>

{::options parse_block_html="false" /}

One with no background slices

<p align="center">
   <a href="https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/tutorials/pizza/blank.png">
<img src="https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/tutorials/pizza/blank.png" style="width:400px">
      </a>
</p>

{::options parse_block_html="true" /}

<details><summary markdown="span">The code</summary>
  
```r
ggplot(df_selected,aes(fct_reorder(Statistic,stat),Percentile)) +                      
  geom_bar(aes(y=100),fill="#F2F4F5",stat="identity",width=1,colour="white",                
           alpha=1,linetype="dashed") +                                                                          
  geom_bar(stat="identity",width=1,fill="#D20222",colour="white") +   
  geom_hline(yintercept=25, colour="white",linetype="longdash",alpha=0.5)+
  geom_hline(yintercept=50, colour="white",linetype="longdash",alpha=0.5)+
  geom_hline(yintercept=75, colour="white",linetype="longdash",alpha=0.5)+ 
  geom_hline(yintercept=100, colour="white",alpha=0.5)+ 
  coord_polar() +                                                                     
  geom_label(aes(label=Per.90),fill="#D20222",size=2,color="white",show.legend = FALSE)+     
  scale_fill_manual(values=c("Possession" = "#D70232",                                  
                             "Attacking" = "#1A78CF",
                             "Defending" = "#FF9300")) +                                                              
  scale_y_continuous(limits = c(-10,100))+                                              
  labs(fill="",   
       caption = "Data from StatsBomb via FBref",     
       #remove legend title
       title=glue("{df_selected$player_name[1]} | Manchester United"),
       subtitle = glue::glue("{df_selected$season} | Compared to midfielders Top 5 competitions | stats per 90"))+                                               
  
  theme_minimal() +                                                                     
  theme(plot.background = element_rect(fill = "#F2F4F5",color = "#F2F4F5"),
        panel.background = element_rect(fill = "#F2F4F5",color = "#F2F4F5"),
        legend.position = "top",
        axis.title.y = element_blank(),
        axis.title.x = element_blank(),
        axis.text.y = element_blank(),
        axis.text.x = element_text(size = 6, angle = ang),
        text = element_text(family="Spartan-Light"),                                    
        plot.title = element_markdown(hjust=0.5,family="Spartan-Medium"),
        plot.subtitle = element_text(hjust=0.5,size=8),
        plot.caption = element_text(hjust=0.5,size=6),
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank(),
        plot.margin = margin(5,2,2,2)) 
```


</details>
<br/>

{::options parse_block_html="false" /}

One with other labels

<p align="center">
   <a href="https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/tutorials/pizza/label.png">
<img src="https://raw.githubusercontent.com/RobinKoetsier/robinkoetsier.github.io/master/assets/img/tutorials/pizza/label.png" style="width:400px">
      </a>
</p>

{::options parse_block_html="true" /}

<details><summary markdown="span">The code</summary>
  
```r
label_data <- df_selected

# calculate the ANGLE of the labels
number_of_bar <- nrow(label_data)
label_data$id <- seq(1,length(label_data$player_name))
angle <-  90 - 360 * (label_data$id-0.5) /number_of_bar     # I substract 0.5 because the letter must have the angle of the center of the bars. Not extreme right(1) or extreme left (0)

# calculate the alignment of labels: right or left
# If I am on the left part of the plot, my labels have currently an angle < -90
label_data$hjust<-ifelse( angle < -90, 1, 0)

# flip angle BY to make them readable
label_data$angle<-ifelse(angle < -90, angle+180, angle)



ggplot(df_selected,aes(fct_reorder(Statistic,stat),Percentile)) +                      
  geom_bar(aes(y=100),fill="#F2F4F5",stat="identity",width=1,colour="#0066B2",                
           alpha=1,linetype="dashed") +                                                                          
  geom_bar(stat="identity",width=1,fill="#CC0033",colour="white") +   
  geom_hline(yintercept=25, colour="white",linetype="longdash",alpha=0.5)+
  geom_hline(yintercept=50, colour="white",linetype="longdash",alpha=0.5)+
  geom_hline(yintercept=75, colour="white",linetype="longdash",alpha=0.5)+ 
  geom_hline(yintercept=100, colour="white",alpha=0.5)+ 
  coord_polar() +                                                                     
  geom_label(aes(label=Per.90),fill="#CC0033",size=2,color="white",show.legend = FALSE)+     
  scale_fill_manual(values=c("Possession" = "#D70232",                                  
                             "Attacking" = "#1A78CF",
                             "Defending" = "#FF9300")) +                                                              
  scale_y_continuous(limits = c(-10,110))+                                              
  labs(fill="",   
       caption = "Data from StatsBomb via FBref",     
       #remove legend title
       title=glue("{df_selected$player_name[1]} | Bayern Munich"),
       subtitle = glue::glue("{df_selected$season} | Compared to attackers Top 5 competitions | stats per 90"))+                                               
  geom_text(data=label_data, aes(x=id, y=100+10, label=Statistic, hjust=hjust), 
      color="#0066B2", fontface="bold",alpha=0.6, size=2.5, angle= label_data$angle, inherit.aes = FALSE )  +
  theme_minimal() +                                                                     
  theme(plot.background = element_rect(fill = "#F2F4F5",color = "#F2F4F5"),
        panel.background = element_rect(fill = "#F2F4F5",color = "#F2F4F5"),
        legend.position = "top",
        axis.title.y = element_blank(),
        axis.title.x = element_blank(),
        axis.text.y = element_blank(),
       # axis.text.x = element_text(size = 6, angle = ang),
        axis.text.x = element_blank(),
        text = element_text(family="Spartan-Light"),                                    
        plot.title = element_markdown(hjust=0.5,family="Spartan-Medium"),
        plot.subtitle = element_text(hjust=0.5,size=8),
        plot.caption = element_text(hjust=0.5,size=6),
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank(),
        plot.margin = margin(5,2,2,2)) 
```


</details>
<br/>

{::options parse_block_html="false" /}


