hw05-tsmith93
================
Thomas Smith
2018-10-15

Assignment structure
--------------------

1.  Overview
2.  Loading packages
3.  Factor management
4.  File I/O
5.  Visualization design
6.  Writing figures to file

### 1. Overview

For this assignment, we will be exploring factors, file input/output, visualization design, and writing figures to files. In order to explore and highlight important aspects of each component, we will walk through examples together.

### 2. Loading packages

If you haven't already done so, download both gapminder and tidyverse using `install.packages()`

Next load gapminder, tidyverse and knitr:

``` r
#suppressPackageStartupMessages stops unecessary messages from popping up
suppressPackageStartupMessages(library(tidyverse))
suppressPackageStartupMessages(library(gapminder))
suppressPackageStartupMessages(library(knitr))
suppressPackageStartupMessages(library(plotly))
```

### 3. Factor management

#### 3.1 Dropping unused levels

##### Create an unused level

For the first part of this assignment, we are going to practice dropping a specific factor from a dataset. As an example, we will be using the gapminder dataset. First, lets see what type of variables are factors in this dataset.

``` r
#we will used sapply to return a matrix of variable classes in gapminder
sapply(gapminder, class) %>% 
  kable()
```

|           | x       |
|-----------|:--------|
| country   | factor  |
| continent | factor  |
| year      | integer |
| lifeExp   | numeric |
| pop       | integer |
| gdpPercap | numeric |

This table shows that both country and continent are factors. We can also use the structure function to assess what are factors in this dataset.

``` r
str(gapminder)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    1704 obs. of  6 variables:
    ##  $ country  : Factor w/ 142 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ continent: Factor w/ 5 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
    ##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
    ##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
    ##  $ pop      : int  8425333 9240934 10267083 11537966 13079460 14880372 12881816 13867957 16317921 22227415 ...
    ##  $ gdpPercap: num  779 821 853 836 740 ...

Again, it is clear that both country and continent are factors. Lets focus on continent

We will use `filter()` to remove all data for Oceania.

``` r
#create an object `no_oceania`, which will be the gapminder dataset with Oceania data removed.
no_oceania <- gapminder %>% 
  filter(continent != "Oceania") 
```

To see if data for Oceania was dropped, we will compare the number of rows in the gapminder dataset, the number of rows for Oceania, and the number of rows in the the no\_oceania dataframe.

``` r
nrow(gapminder)
```

    ## [1] 1704

``` r
nrow(gapminder[gapminder$continent == "Oceania",])
```

    ## [1] 24

``` r
nrow(no_oceania)
```

    ## [1] 1680

The total number of rows in gapminder is 1704, and Oceania provides 24 rows, therfore 1680 rows in no\_ocean confirms that data for Oceania has been removed.

Let's look at the structure of `no_oceania` to see if there are any unused levels in the dataset. Specifically, we will be looking at the number of levels for continent.

``` r
str(no_oceania)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    1680 obs. of  6 variables:
    ##  $ country  : Factor w/ 142 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ continent: Factor w/ 5 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
    ##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
    ##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
    ##  $ pop      : int  8425333 9240934 10267083 11537966 13079460 14880372 12881816 13867957 16317921 22227415 ...
    ##  $ gdpPercap: num  779 821 853 836 740 ...

It appears that there are still 5 levels for the continent variable. Lets use a figure to see what is going on.

``` r
#plot continents for no_oceania
ggplot(no_oceania, aes(continent)) +
  geom_bar() +
  #FALSE will plot all existing levels, even if they are empty
  scale_x_discrete(drop = FALSE) + 
  #lets give the titles nice looking names
  ylab("Number of countries") +
  xlab("Continent") +
  ggtitle("Country count per continent") +
  #add the classic theme
  theme_classic()
```

![](hw05-tsmith93_files/figure-markdown_github/unnamed-chunk-8-1.png)

It is very clear to see that Oceania is present as an unused level.

Alternatively to the process just described, we can also easily use fct\_count() to see if Oceania is an unused level in `no_oceania`:

``` r
#fct_count will count all the factors
fct_count(no_oceania$continent) %>% 
#lets present it in a nice table with kable()  
  kable()
```

| f        |    n|
|:---------|----:|
| Africa   |  624|
| Americas |  300|
| Asia     |  396|
| Europe   |  360|
| Oceania  |    0|

##### Drop unused level

Now that Oceania has been identified as an unused, lets use the `droplevels()` function to drop it!

``` r
#make a new object that excludes unused levels
oceania_gone <- droplevels(no_oceania)
```

Now lets check to see whether it has actually been dropped with use graphical representation.

``` r
#again we will plot a bar graph of continents
ggplot(oceania_gone, aes(continent)) +
  geom_bar() +
#again FALSE will plot all existing levels, even if they are empty
  scale_x_discrete(drop = FALSE) +
#add cleaner titles  
  ylab("Number of countries") +
  xlab("Continent") +
  ggtitle("Country count per continent") +
  theme_classic()
```

![](hw05-tsmith93_files/figure-markdown_github/unnamed-chunk-11-1.png)

Ta da! Oceania is no longer present as an unused level.

Again, we can double check with `fct_count()`:

``` r
fct_count(oceania_gone$continent) %>% 
#present factor data in a nice table!  
  kable()
```

| f        |    n|
|:---------|----:|
| Africa   |  624|
| Americas |  300|
| Asia     |  396|
| Europe   |  360|

There you go, the unused level "Oceania" is confirmed to be gone.

#### 3.2 Reorder factors

##### Look at default order

In this section, we will use the packages `forcats` and `ggplot2` (part of tidyverse) in order to reorder levels in the gapminder dataset so that data can be displayed in a more user friendly, or user specific, way.

For this example, we will look at life expectancy in each continent. Specifically, we will do a jitter plot and have the minimum and maximum values highlighted.

``` r
#plot life expectancies for each continent
ggplot(gapminder, aes(x = continent, y = lifeExp)) +
#add a log10 scale to the y axis
  scale_y_log10() +
#use jitter to understand the data spread, change the aesthetics of the jitter points
  geom_jitter(position = position_jitter(width = 0.1, height = 0), alpha = 1/4) +
#plot all of the minimum values with blue points  
  stat_summary(fun.y = min, colour = "blue", geom = "point", size = 5) +
#plot all of the minimum values with orange points   
  stat_summary(fun.y = max, colour = "orange", geom = "point", size = 5) +
#give the labels nice looking titles  
  ylab("Life expectancy") +
  xlab("Continent") +
  ggtitle("Life expectancy for each continent (1952 - 2007)") +
#specify the theme of figure, add white background, remove grid, make the axis lines black  
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(), panel.background = element_blank(),
  axis.line = element_line(colour = "black"))
```

![](hw05-tsmith93_files/figure-markdown_github/unnamed-chunk-13-1.png)

In this figure, we can see that the continents are ordered alphabetically. But what if we want them ordered differently?

##### Reorder

Factors can be reorder so that data is displayed in a more desired way. Here are a couple examples!

Let's try to order the continents based on the ascending minimum values for life expectancy.

``` r
#lets make a new object with continent and life expectancy with factors re-ordered basd on minimum life expectancy  
gap_lifeExp_min_asc <- gapminder %>% 
  mutate(continent = fct_reorder(continent, lifeExp, .fun = min))

#lets plot this new object the same way we plotted the last figure
ggplot(gap_lifeExp_min_asc, aes(x = continent, y = lifeExp)) +
#add log10 scale  
  scale_y_log10() +
#use jitter to display spread  
  geom_jitter(position = position_jitter(width = 0.1, height = 0), alpha = 1/4) +
#highlight minimum and maximum values  
  stat_summary(fun.y = min, colour = "blue", geom = "point", size = 5) +
  stat_summary(fun.y = max, colour = "orange", geom = "point", size = 5) +
#add nice looking titles  
  ylab("Life expectancy") +
  xlab("Continent") +
  ggtitle("Life expectancy for each continent (1952 -2007)") +
#change them so background is blank and white, with black axes  
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(), panel.background = element_blank(),
  axis.line = element_line(colour = "black"))
```

![](hw05-tsmith93_files/figure-markdown_github/unnamed-chunk-14-1.png)

Look at that! It is very clear Africa had the lowest minimum life expectancy and Oceania had the highest minimum life expectancy. Lets try switching that order so that it is done by descending minimum values.

``` r
#assign the reorder factor data to a new object
gap_lifeExp_min_des <- gapminder %>% 
  mutate(continent = fct_reorder(continent, lifeExp, .fun = min, .desc = TRUE))

#plot the same figure for consistency
ggplot(gap_lifeExp_min_des, aes(x = continent, y = lifeExp)) +
#add log10 scale  
  scale_y_log10() +
#use jitter so all dat points are seen - spread is clearly shown!  
  geom_jitter(position = position_jitter(width = 0.1, height = 0), alpha = 1/4) +
#minimum and maximum life expectancy valeus highlighted  
  stat_summary(fun.y = min, colour = "blue", geom = "point", size = 5) +
  stat_summary(fun.y = max, colour = "orange", geom = "point", size = 5) +
#clean looking titles  
  ylab("Life expectancy") +
  xlab("Continent") +
  ggtitle("Life expectancy for each continent (1952 - 2007)") +
#white background and black axes -look at past figures for more in depth explanation :)  
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(), panel.background = element_blank(),
  axis.line = element_line(colour = "black"))
```

![](hw05-tsmith93_files/figure-markdown_github/unnamed-chunk-15-1.png)

There we go, same information shown as the last figure, but the order of factors has been reversed!

### Part 2: File I/O

In this section, we are going to see what happens when we save data to a csv file and reopen that csv file. Specifically, we will look to see if reordering of factors is maintained. First lets make a new data object.

#### 2.1 Make new object

``` r
#lets make a new object with data from Asia only
asia <- gapminder %>% 
  filter(continent == "Asia") 

#plot life expectancies for each country
asia %>% 
  ggplot(aes(country, lifeExp)) +
  geom_point() +
#this will flip axes so labels are read more easily 
  coord_flip() +
#give the figure nice titles  
  labs(title = "Life expectancy in Asia (1957-2007)",
       x = "Life expectancy (years)",
       y = "Country") +
#give figure a nice look  
  theme_light()
```

![](hw05-tsmith93_files/figure-markdown_github/unnamed-chunk-16-1.png)

The figure looks a little cluttered. We will reorder factors to make it look nicer.

#### 2.2 Reorder factors

``` r
#new object with factors (countries in this case) ordered by median
asia_reorder <- asia %>% 
mutate(country = fct_reorder(country, lifeExp, .fun = median))

asia_reorder %>% 
#plot country and life expectancy  
  ggplot(aes(country, lifeExp)) +
  geom_point() +
#this will flip axes so labels are read more easily   
  coord_flip() + 
#ncie to read titles  
  labs(title = "Life expectancy in Asia (1957-2007)",
       x = "Life expectancy (years)",
       y = "Country") +
#give plot a nice lookign theme  
  theme_light()
```

![](hw05-tsmith93_files/figure-markdown_github/unnamed-chunk-17-1.png)

There you can see the figure is much easier to look at, with Afghanistan having the lowest median life expectanyc and Japan havign the highest meanlife expectancy.

#### 2.3 Save data to csv and read the same csv

For this, we will use `write_csv()` and `gap_read()`

``` r
write_csv(asia_reorder, "asia_reorder.csv")

#Reading new data set back in
(gap_read <- read_csv("asia_reorder.csv"))
```

    ## Parsed with column specification:
    ## cols(
    ##   country = col_character(),
    ##   continent = col_character(),
    ##   year = col_integer(),
    ##   lifeExp = col_double(),
    ##   pop = col_integer(),
    ##   gdpPercap = col_double()
    ## )

    ## # A tibble: 396 x 6
    ##    country     continent  year lifeExp      pop gdpPercap
    ##    <chr>       <chr>     <int>   <dbl>    <int>     <dbl>
    ##  1 Afghanistan Asia       1952    28.8  8425333      779.
    ##  2 Afghanistan Asia       1957    30.3  9240934      821.
    ##  3 Afghanistan Asia       1962    32.0 10267083      853.
    ##  4 Afghanistan Asia       1967    34.0 11537966      836.
    ##  5 Afghanistan Asia       1972    36.1 13079460      740.
    ##  6 Afghanistan Asia       1977    38.4 14880372      786.
    ##  7 Afghanistan Asia       1982    39.9 12881816      978.
    ##  8 Afghanistan Asia       1987    40.8 13867957      852.
    ##  9 Afghanistan Asia       1992    41.7 16317921      649.
    ## 10 Afghanistan Asia       1997    41.8 22227415      635.
    ## # ... with 386 more rows

``` r
#We will display this data using the same code as before
(gap_read %>% 
  ggplot(aes(country, lifeExp)) +
  geom_point() +
#make the labels on axes easier to read
  coord_flip() + 
#give nice axis titles    
  labs(title = "Life expectancy in Asia (1957-2007)",
       x = "Life expectancy (years)",
       y = "Country") +
  theme_light())
```

![](hw05-tsmith93_files/figure-markdown_github/unnamed-chunk-18-1.png)

It is very clear that the reorderign of factors is not maintained when you save a csv.

#### 2.4 Save to RDS and read RDS

Here we will use `saveRDS` to save asia\_reorder to RDS. Immediately after we will read the RDS. Finally, we will compare the two to see if they are identical.

``` r
#have object, and include pathway to where you want to export
saveRDS(asia_reorder, '/Users/thomassmith/Documents/MSc courses/STAT 545A/Assignment/hw05-tsmith93/hw05-tsmith93/asia_reorder.rds')

#assign RDS data to an object. Specify the file pathway
asia_reorder_RDS <- readRDS('/Users/thomassmith/Documents/MSc courses/STAT 545A/Assignment/hw05-tsmith93/hw05-tsmith93/asia_reorder.rds') 

#look to see if the files are the same
identical(asia_reorder, asia_reorder_RDS) 
```

    ## [1] TRUE

Look at that, the exported RDS is identical the imported RDS.

### Part 3: Visualization design

For this section, we will getting more practice with plot visualization. Specifically, we will be using both ggplot2, as well as plotly. This will allow us to make comparisons between the two data visualization packages.

#### 3.1 Basic plots

As an example, we will practice with the `gapminder` dataset, looking at GDP and life expectancy for Scandinavian countries.

``` r
#Create an object called 'scandinavia' to store the desired data 
(scandinavia <- 
  #call upon the gapminder dataset
  gapminder %>%
  #Filter so that we are only using Canada, Russia, Japan, and United States
  filter(country=="Finland" | country == "Norway" | country == "Sweden" | country == "Iceland" | country == "Denmark") %>% 
  #Mae table cleaner by rounding the life expectancies off the nearest integer
  mutate(lifeExp = round(lifeExp, 0)) %>% 
  #Select the columns country, year and life expectancy
  select(country, year, lifeExp, gdpPercap))
```

    ## # A tibble: 60 x 4
    ##    country  year lifeExp gdpPercap
    ##    <fct>   <int>   <dbl>     <dbl>
    ##  1 Denmark  1952      71     9692.
    ##  2 Denmark  1957      72    11100.
    ##  3 Denmark  1962      72    13583.
    ##  4 Denmark  1967      73    15937.
    ##  5 Denmark  1972      73    18866.
    ##  6 Denmark  1977      75    20423.
    ##  7 Denmark  1982      75    21688.
    ##  8 Denmark  1987      75    25116.
    ##  9 Denmark  1992      75    26407.
    ## 10 Denmark  1997      76    29804.
    ## # ... with 50 more rows

Lets first make a basic plot of life expectancy over time for these countries:

``` r
#call upon object 'scandinavia'
(scandinavia %>% 
  #use ggplot with 'year' and 'lifeExp' as x and y
  ggplot(aes(year, lifeExp)) +
  #add a scatterplot with point colour specified by country
  geom_point(aes(colour= country)) +
  #add a title
  ggtitle("Life Expectancies in Scandinavia (1952-2007)") +
  geom_smooth(method = lm, se = FALSE, aes(colour = country)))
```

![](hw05-tsmith93_files/figure-markdown_github/unnamed-chunk-21-1.png)

I think we can make the figure look much cleaner with a few changes to the code.

``` r
#call upon object 'scandinavia'
scandinavia %>% 
  #use ggplot with 'year' and 'lifeExp' as x and y
  ggplot(aes(year, lifeExp)) +
  #add a scatterplot with point colour specified by country
  geom_point(aes(colour= country)) +
  #add a title
  ggtitle("Life Expectancies in Scandinavia (1952-2007)") +
  geom_smooth(method = lm, se = FALSE, aes(colour = country)) +
  #add x and y labels
  xlab("Year") +
  ylab("Life expectancy (years)") +
  #clean up the aesthetic of the plot!
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(), panel.background = element_blank(),
  axis.line = element_line(colour = "black")) 
```

![](hw05-tsmith93_files/figure-markdown_github/unnamed-chunk-22-1.png)

Wow, that thing is way nicer to look at! Lets do the same plot, but this time use GDP per capita on the y axis.

``` r
#call upon object 'scandinavia'
scandinavia_plot <- scandinavia %>% 
  #use ggplot with 'year' and 'lifeExp' as x and y
  ggplot(aes(year, gdpPercap)) +
  #add a scatterplot with point colour specified by country
  geom_point(aes(colour= country)) +
  #add a title
  ggtitle("GDP per capita in Scandinavia (1952-2007)") +
  geom_smooth(method = lm, se = FALSE, aes(colour = country)) +
  #add x and y labels
  xlab("Year") +
  ylab("GDP per capita") +
  #clean up the aesthetic of the plot!
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(), panel.background = element_blank(),
  axis.line = element_line(colour = "black")) 

#display the plot
scandinavia_plot
```

![](hw05-tsmith93_files/figure-markdown_github/unnamed-chunk-23-1.png)

Wow, another great looking plot. But I think we can display the data from these two plots in one figure. To do this, we will have to use `plotly` package, which we loaded at the very beginning of this file!

#### 3.2 plotly figures

Warning: As plotly figures seem to embed oddly in github, please look at [this](https://plot.ly/~tsmith93/5/#/) link for my plotly. I have also commented out the code so that this page is easier for you to scroll through.

``` r
#select scandinavia data
#(three_axes <- plot_ly(scandinavia, 
#assign a variable to the x, y, and z axes        
        #x = ~year, 
        #y = ~lifeExp, 
        #z = ~gdpPercap,
#select the type of plotly plot
        #type = "scatter3d",
#specify the type of markers and look of them
        #mode = "markers",
        #opacity = 0.2))
```

What an efficient way of putting all the data together. Try moving the figure around with your mouse! Its interactive!

#### 3.3 Convert a ggplot to plotly

We will convert it to a plotly graph using `ggplotly()`. As this appears as huge chunks of code, we will describe the differences now. With ggplot, you are given basic figures. With plotly, you are able to be much more interactive with your plots. Yuo can zoom in and zoom out, and look at what the exact values are for points by hoverign your mouse over them.

``` r
#Again, we will comment out this for the users benefit!
#Converting previous plot to plotly figure
#(scandinavia_plotly <- ggplotly(scandinavia_plot))
```

### Part 4: Writing figures to file

Here we will use `ggsave()` to make image files. We will explore changing the content of the images, as well as the file type

#### 4.1 Save to .jpg

``` r
ggsave("scandinavia_plot.jpg", scandinavia_plot)
```

    ## Saving 7 x 5 in image

![scandinavia plot](https://github.com/STAT545-UBC-students/hw05-tsmith93/blob/master/scandinavia_plot.png?raw=true)

Lets make it look nicer!

``` r
ggsave("scandinavia_plot_nice.jpg", plot = scandinavia_plot, device = "jpg", width = 6, height = 4, units = "in", dpi = 300)
```

![fancy scandinavia plot](https://github.com/STAT545-UBC-students/hw05-tsmith93/blob/master/scandinavia_plot_nice.jpg?raw=true)

#### 4.2 Save to .png

``` r
ggsave("scandinavia_plot.png", scandinavia_plot)
```

    ## Saving 7 x 5 in image

#### 4.3 Save to .pdf

``` r
ggsave("scandinavia_plot.pdf", scandinavia_plot)
```

    ## Saving 7 x 5 in image
