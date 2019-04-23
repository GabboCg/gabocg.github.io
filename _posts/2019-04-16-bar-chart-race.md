---
title: "Bar Chart Races With gganimate"
author: Emily E. Kuehler
post_date: 2019-04-16
permalink: /bar-chart-race/
categories: tidytuesday, gganimate
output: jekyllthat::jekylldown
excerpt_separator: <!--more-->
---

![Alt Text](/img/barplot_race.gif)

Grand Slam Tennis Bar Chart Race
--------------------------------

This week's Tidy Tuesday dataset contained data about Grand Slam Tennis Tournaments during the Open Era. [John Burn-Murdoch](https://twitter.com/jburnmurdoch) from The Financial Times had put together a series of beautiful visualizations using a similar dataset, which reminded me of the [bar chart races](https://observablehq.com/@johnburnmurdoch/bar-chart-race-the-most-populous-cities-in-the-world) he made in D3. I had been wanting to see if I could replicate something similar with gganimate so decided to give it a try.

### Data Wrangling

First, we'll load the necessary packages and read in our data.

``` r
library(tidyverse); library(gganimate)

grand_slams <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-04-09/grand_slams.csv")
```

Next, let's take a look at the structure of the data.

``` r
head(grand_slams)
```

    ## # A tibble: 6 x 6
    ##    year grand_slam     name         rolling_win_cou… tournament_date gender
    ##   <dbl> <chr>          <chr>                   <dbl> <date>          <chr> 
    ## 1  1968 australian_op… Billie Jean…                1 1968-01-10      Female
    ## 2  1968 french_open    Nancy Richey                1 1968-06-09      Female
    ## 3  1968 wimbledon      Billie Jean…                2 1968-07-14      Female
    ## 4  1968 us_open        Virginia Wa…                1 1968-09-09      Female
    ## 5  1969 australian_op… Margaret Co…                1 1969-01-10      Female
    ## 6  1969 french_open    Margaret Co…                2 1969-06-09      Female

So we have a tibble with six variables:

`year`: year in which the grand slam was played `grand_slam`: the name of the tournament `name`: champion of corresponding tournament `rolling_win_count`: rolling count of grand slam wins for corresponding player `tournament_date`: approximate tournament date `gender`: player gender

In order to prep the data for our bar chart race in gganimate, we need to group the data by each point in time and find players with the top 10 rolling win count for each point in time. We define a point in time as the completion of a grand slam tournament. Also, for this graphic, we will construct a bar chart of the top 10 wins from all players, rather than separating by gender.

For the first few years in the dataset, the scant data at this point in time results in either a very sparse bar chart or a bar chart filled with ties at the bottom. Because of this, I chose to start the bar chart race in 1975, when there was enough data to create a full top 10 bar chart without a ton of ties at the bottom, which would distort the shape of the graph.

The first step we'll take is to give a numeric ordering to the tournaments and arrange the data by tournament date.

``` r
grand_slams_clean <- grand_slams %>% 
  mutate(tournament_order = case_when(grand_slam=='australian_open' ~ 1,
                                      grand_slam=='french_open' ~ 2,
                                      grand_slam=='wimbledon' ~ 3,
                                      grand_slam=='us_open' ~ 4)) %>%
  arrange(tournament_date)

head(grand_slams_clean)
```

    ## # A tibble: 6 x 7
    ##    year grand_slam name  rolling_win_cou… tournament_date gender
    ##   <dbl> <chr>      <chr>            <dbl> <date>          <chr> 
    ## 1  1968 australia… Bill…                1 1968-01-10      Female
    ## 2  1968 australia… Will…                1 1968-01-10      Male  
    ## 3  1968 french_op… Nanc…                1 1968-06-09      Female
    ## 4  1968 french_op… Ken …                1 1968-06-09      Male  
    ## 5  1968 wimbledon  Bill…                2 1968-07-14      Female
    ## 6  1968 wimbledon  Rod …                1 1968-07-14      Male  
    ## # … with 1 more variable: tournament_order <dbl>

Next, we'll go through a two step process to ultimately prepare our data for gganimate. As noted before, we are going to start in 1975 because of the multiple ties that existed in previous years. The grouping for 1975 will consist of the players with the top 10 rolling wins from 1968 through 1975 (i.e. all the tournaments of 1975). We will label this `init_df`. After this each grouping will be a point in time, which is after a single grand slam tournament.

``` r
#get data from 1968-1975, helps avoid ties, incomplete bar chart at beginning
#basically just making this more visually appealling
init_df <- grand_slams_clean %>%
  filter(year <= 1975) %>%
  group_by(name) %>%
  filter(rolling_win_count==max(rolling_win_count)) %>%
  ungroup() %>%
  top_n(10, wt=rolling_win_count) %>%
  arrange(desc(rolling_win_count)) %>%
  select(name,gender, rolling_win_count) %>%
  mutate(curr_year = 1975,
         ordering = as.double(rev(seq(10:1))) * 1.0)

#outer loop gets year
for (i in 1976:2019) {
  #inner loop gets tournament
  for (j in 1:4) {
    tmp_df <- grand_slams_clean %>%
      #filter data up to correct point in time
      filter(year < i | (year==i & tournament_order <= j)) %>%
      #get each players max win count
      group_by(name) %>% 
      filter(rolling_win_count==max(rolling_win_count)) %>% 
      ungroup() %>% 
      top_n(10, wt=rolling_win_count) %>%
      select(name, gender, rolling_win_count) %>%
      arrange(desc(rolling_win_count)) %>%
      slice(1:10) %>%
      #add var for curr_year, ordering for easy bar chart (reverse it cuz we're gonna do horiz)
      mutate(curr_year = i,
             tournament_num = j,
             ordering = as.double(rev(seq(10:1))) * 1.0) 
    init_df <- init_df %>%
      bind_rows(tmp_df)
  }
}

head(init_df)
```

    ## # A tibble: 6 x 6
    ##   name            gender rolling_win_cou… curr_year ordering tournament_num
    ##   <chr>           <chr>             <dbl>     <dbl>    <dbl>          <int>
    ## 1 Margaret Court  Female               11      1975       10             NA
    ## 2 Billie Jean Ki… Female                9      1975        9             NA
    ## 3 Rod Laver       Male                  5      1975        8             NA
    ## 4 John Newcombe   Male                  5      1975        7             NA
    ## 5 Ken Rosewall    Male                  4      1975        6             NA
    ## 6 Evonne Goolago… Female                4      1975        5             NA

So this is starting to look pretty good. However, when we put the data into gganimate, we want each frame to transition after a grand slam tournament. Right now, we have the variables `curr_year`, however there are four tournaments in a year, so that transition is too long, and `tournament_num`, which identifies tournaments correctly, but not uniquely. But, we can use these variables to create a unique id and easily plug that into gganimate.

``` r
final_df <- init_df %>% 
  group_by(curr_year, tournament_num) %>% 
  mutate(frame_id = group_indices()) %>% 
  ungroup() %>% 
  head()
```

Before diving into our plot, let's take a look at player names, which we're going to be using as labels on the bars.

``` r
unique(final_df$name)
```

    ## [1] "Margaret Court"          "Billie Jean King"       
    ## [3] "Rod Laver"               "John Newcombe"          
    ## [5] "Ken Rosewall"            "Evonne Goolagong Cawley"

Here we see `Evonne Goolagong Cawley` is VERY long. During her playing career she went by `Evonne Goolagong` so I don't feel terrible about dropping her married name for labeling ease. `Martina Navratilova` is also quite long, but that's just gonna stay.

``` r
final_df <- final_df %>% mutate(name = ifelse(name == 'Evonne Goolagong Cawley', 'Evonne Goolagong', name))
```

### The Fun Part!

Ok, now that we have our data in the right format, we can go ahead and make our plot. The first thing we'll do is set our theme, palette and font.

``` r
my_font <- 'Quicksand'
my_background <- 'antiquewhite'
my_pal <- c('#F8AFA8','#74A089') #colors for bars (from wesanderson)
my_theme <- my_theme <- theme(text = element_text(family = my_font),
                              rect = element_rect(fill = my_background),
                              plot.background = element_rect(fill = my_background, color = NA),
                              panel.background = element_rect(fill = my_background, color = NA),
                              panel.border = element_blank(),
                              plot.title = element_text(face = 'bold', size = 20),
                              plot.subtitle = element_text(size = 14),
                              panel.grid.major.y = element_blank(),
                              panel.grid.minor.y = element_blank(),
                              panel.grid.major.x = element_line(color = 'grey75'),
                              panel.grid.minor.x = element_line(color = 'grey75'),
                              legend.position = 'none',
                              plot.caption = element_text(size = 8),
                              axis.ticks = element_blank(),
                              axis.text.y =  element_blank())

theme_set(theme_light() + my_theme)
```

Ultimately we're going to have two bar colors, one for male players and one for female players and then otherwise set a relatively minimal theme (doing my best to replicate [John Burn-Murdoch's](https://observablehq.com/@johnburnmurdoch/bar-chart-race-the-most-populous-cities-in-the-world) design, though with a different color scheme).

#### Bar Chart Race Attempt 1

So the question for the bar chart race in gganimate was how to make the transitions between time periods smooth. Thankfully, I found this [EXTREMELY helpful stackoverflow post](https://stackoverflow.com/questions/52623722/how-does-gganimate-order-an-ordered-bar-time-series/52652394#52652394) that directed me toward the use of geom\_tile() and saved a lot of time that I probably would have otherwise wasted testing out geom\_bar().

``` r
barplot_race_blur <- ggplot(aes(ordering, group = name), data = final_df) +
  geom_tile(aes(y = rolling_win_count / 2, 
                height = rolling_win_count,
                width = 0.9, fill=gender), alpha = 0.9) +
  scale_fill_manual(values = my_pal) +
  geom_text(aes(y = rolling_win_count, label = name), family=my_font, nudge_y = -2, size = 3) +
  geom_text(aes(y = rolling_win_count, label = rolling_win_count), family=my_font, nudge_y = 0.5) +
  geom_text(aes(x=1,y=18.75, label=paste0(curr_year)), family=my_font, size=8, color = 'gray45') +
  coord_cartesian(clip = "off", expand = FALSE) +
  coord_flip() +
  labs(title = 'Most Grand Slam Singles Championships',
       subtitle = 'Open Era Only',
       caption = 'data source: Wikipedia | plot by @emilykuehler',
       x = '',
       y = '') +
  transition_states(frame_id, 
                    transition_length = 4, state_length = 3) +
  ease_aes('cubic-in-out')
```

![Alt Text](/img/barplot_race_blur.gif)

So this animation is generally doing what we want, with one huge exception. During the transitions, we see trailing decimals on the labels, which looks awful. My first attempt at fixing this was to use various rounding strategies, such as flooring the win count. This did not work. What we have to do is change the data type of the label to character in order to get rid of the trailing decimals.

### Final Plot

``` r
barplot_race_blur <- ggplot(aes(ordering, group = name), data = final_df) +
  geom_tile(aes(y = rolling_win_count / 2, 
                height = rolling_win_count,
                width = 0.9, fill=gender), alpha = 0.9) +
  scale_fill_manual(values = my_pal) +
  geom_text(aes(y = rolling_win_count, label = name), family=my_font, nudge_y = -2, size = 3) +
  #convert to character to get rid of blurry trailing decimals
  geom_text(aes(y = rolling_win_count, label = as.character(rolling_win_count)), family=my_font, nudge_y = 0.5) +
  geom_text(aes(x=1,y=18.75, label=paste0(curr_year)), family=my_font, size=8, color = 'gray45') +
  coord_cartesian(clip = "off", expand = FALSE) +
  coord_flip() +
  labs(title = 'Most Grand Slam Singles Championships',
       subtitle = 'Open Era Only',
       caption = 'data source: Wikipedia | plot by @emilykuehler',
       x = '',
       y = '') +
  transition_states(frame_id, 
                    transition_length = 4, state_length = 3) +
  ease_aes('cubic-in-out')
```

![Alt Text](/img/barplot_race.gif)

Sweet, this is what we want!

This was my first time working with gganimate and in addition to the posts already mentioned, I found [this tutorial](https://github.com/ropenscilabs/learngganimate/blob/2872425f08392f9f647005eb19a9d4afacd1ab44/animate.md#saving-your-animation) by [Will Chase](https://twitter.com/W_R_Chase) to be invaluable. It looks like there are many contributors to what is a pretty big repository of fun stuff, so I am quite excited to explore it further.

### One More For The Road

I liked the tennis animation a lot, but bar chart races are a little bit more fun when there is more growth between time periods (for grand slam tennis, the most a player can win is a single slam). So I also made one that tracks the all-time leading scorers in the NBA over time. Not gonna go through code for this one as it's quite similar, but you can check it out [here](https://github.com/emilykuehler/bar-chart-race/blob/master/nba_all_time_leading_scorers.R) if you'd like.

![Alt Text](/img/nba_scoring_leaders.gif)
