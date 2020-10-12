FOO\_plots\_terns
================
Gemma Clucas
9/29/2020

### Read in the rarefied feature table

Note, I made a copy of the xlsx file and saved it as a csv file. I also
deleted the full taxonomy strings, keeping only the common names of the
fish species.

Read in the csv and change the `Year` variable to a factor. Also correct
spelling
errors.

``` r
df <- read.csv("/Users/gemmaclucas/Dropbox/Diets_from_poop/2019_terns_puffins_fecal_data_analysis/MiFish/final_taxonomy_superblast/Terns/Terns_table_rarefied400_withtaxonomyedits.csv", header = TRUE) %>% 
  mutate(Year = as.factor(Year)) %>% 
  rename(River.herring = River.Herring,
         Sandlance = Sandlances,
         Mummichog = Mummichig)
```

### Create plotting order

I need to define the order of the fish along the x-axis, so that it is
consistent among graphs.

``` r
# Define x axis order
Species_ordered <-  c("Atlantic herring",
                      "River herring",
                      "Sandlance",
                      "White hake",
                      "Silver hake",
                      "Red hake",
                      "Fourbeard rockling",
                      "Haddock",
                      "Atlantic mackerel",
                      "Atlantic butterfish",
                      "Cunner",
                      "Acadian redfish",
                      "Mummichog",
                      "Atlantic silverside",
                      "Atlantic tomcod",
                      "Spotted codling",
                      "Three spined stickleback",
                      "Radiated shanny",
                      "American angler",
                      "Atlantic cod",
                      "Black spotted stickleback",
                      "Nine spine stickleback",
                      "Saithe",
                      "Tautog",
                      "Darter sp",
                      "Red lionfish")

# Define colour palette - HAVEN'T FINISHED AS THIS WILL CHANGE DEPENDING ON PLOTTING ORDER
Species_colours <- c(rgb(98, 64, 18, max = 255),
                     rgb(150,   102,    34  , max = 255),
                     rgb(207,   138,    54  , max = 255),
                     rgb(247,   224,    167 , max = 255),
                     rgb(225,   204,    150 , max = 255),
                     rgb(187,   169,    126 , max = 255),
                     rgb(214,   237,    234 , max = 255),
                     rgb(159,   211,    204 , max = 255),
                     rgb(92,    164,    160 , max = 255),
                     rgb(50,    118,    113     , max = 255),
                     rgb(30,    75,  63     , max = 255),
                     rgb(5, 60, 245     , max = 255),
                     rgb(5, 57, 237     , max = 255),
                     rgb(6, 56, 234     , max = 255),
                     rgb(3, 46, 199     , max = 255),
                     rgb(150,   102,    34  , max = 255),
                     rgb(150,   102,    34  , max = 255),
                     rgb(150,   102,    34  , max = 255),
                     rgb(150,   102,    34  , max = 255),
                     rgb(150,   102,    34  , max = 255),
                     rgb(150,   102,    34  , max = 255),
                     rgb(150,   102,    34  , max = 255),
                     rgb(150,   102,    34  , max = 255),
                     rgb(150,   102,    34  , max = 255),
                     rgb(150,   102,    34  , max = 255),
                     rgb(150,   102,    34  , max = 255))

Species_with_colours <- c("Atlantic herring" = rgb(98, 64, 18, max = 255),
                      "River herring" = rgb(150,    102,    34  , max = 255),
                      "Sandlance" = rgb(207,    138,    54  , max = 255),
                      "White hake" = rgb(247,   224,    167 , max = 255),
                      "Silver hake" = rgb(225,  204,    150 , max = 255),
                      "Red hake" = rgb(187, 169,    126 , max = 255),
                      "Fourbeard rockling" = rgb(214,   237,    234 , max = 255),
                      "Haddock" = rgb(159,  211,    204 , max = 255),
                      "Atlantic mackerel" = rgb(92, 164,    160 , max = 255),
                      "Atlantic butterfish" = rgb(50,   118,    113     , max = 255),
                      "Cunner" = rgb(30,    75,  63     , max = 255),
                      "Acadian redfish" = rgb(5,    60, 245     , max = 255),
                      "Mummichog" = rgb(5,  57, 237     , max = 255),
                      "Atlantic silverside" = rgb(6,    56, 234     , max = 255),
                      "Atlantic tomcod" = rgb(3,    46, 199     , max = 255),
                      "Spotted codling" = rgb(150,  102,    34  , max = 255),
                      "Three spined stickleback" = rgb(150, 102,    34  , max = 255),
                      "Radiated shanny" = rgb(150,  102,    34  , max = 255),
                      "American angler" = rgb(150,  102,    34  , max = 255),
                      "Atlantic cod" = rgb(150, 102,    34  , max = 255),
                      "Black spotted stickleback" = rgb(150,    102,    34  , max = 255),
                      "Nine spine stickleback" = rgb(150,   102,    34  , max = 255),
                      "Saithe" = rgb(150,   102,    34  , max = 255),
                      "Tautog" = rgb(150,   102,    34  , max = 255),
                      "Darter sp" = rgb(150,    102,    34  , max = 255),
                      "Red lionfish" = rgb(150, 102,    34  , max = 255))
```

### Functions to calculate FOO from filtered feature table and plot FOO

I am going to filter the feature table for e.g. COTE adults from 2017 by
hand, but then I just want to be able to use functions to calculate FOO
and plot, to save on the amount of code I have to write.

``` r
calc_FOO <- function(x) {
  x %>% 
  mutate_if(is.numeric, ~1 * (. > 0)) %>%     # change to detection/non-detection
  summarise_each(funs = sum) %>%              # count number of detections
  melt() %>%                                  # make into long dataframe
  rename(Occurrence = value,
         Species = variable) %>% 
  mutate(FOO = Occurrence/n_samples*100)
}


# get rid of periods in column names and order species
scrub_periods <- function(x) {
  x$Species <-  gsub("\\.", " ", x$Species)
  x$Species <- factor(x$Species, levels = Species_ordered)
}

# function to plot, with the species in the order determined by Species_ordered
plot_FOO <- function(x) {
  x %>% 
    ggplot() +
    geom_bar(aes(x = Species, y = FOO, fill = Species), stat = "identity") +
    theme_classic() +
    theme(axis.text.x = element_text(angle = 45,  hjust=1)) +
    labs(y = "Frequency of occurrence (%)") +
    scale_fill_manual(values = Species_colours,
                      breaks = Species_ordered,
                      labels = Species_ordered)
}
```

### Common Tern chicks in 2017

Select the COTE chick data using `filter`, change to presence/absence
data using the `mutate_if` term, then `melt` to calculate the frequence
of occurrence.

``` r
# Filter the data and save to object
COTE_chicks_2017 <- df %>% 
  filter(Species == "COTE" & Age == "chick" & Year == 2017) %>% 
  select(River.herring:White.hake) 

# record number of records
n_samples <- nrow(COTE_chicks_2017)

FOO <- calc_FOO(COTE_chicks_2017)

FOO$Species <- scrub_periods(FOO)

FOO %>% 
  filter(FOO > 0) %>% 
  plot_FOO()
```

![](FOO_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

### Common Tern chicks in 2018

``` r
# Filter the data and save to object
COTE_chicks_2018 <- df %>% 
  filter(Species == "COTE" & Age == "chick" & Year == 2018) %>% 
  select(River.herring:White.hake) 

# record number of records
n_samples <- nrow(COTE_chicks_2018)

FOO <- calc_FOO(COTE_chicks_2018)

FOO$Species <- scrub_periods(FOO)

FOO %>% 
  filter(FOO > 0) %>% 
  plot_FOO()
```

![](FOO_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

### Roseate Tern chicks in 2017

``` r
# Filter the data and save to object
ROST_chicks_2017 <- df %>% 
  filter(Species == "ROST" & Age == "chick" & Year == 2017) %>% 
  select(River.herring:White.hake) 

# record number of records
n_samples <- nrow(ROST_chicks_2017)

FOO <- calc_FOO(ROST_chicks_2017)

FOO$Species <- scrub_periods(FOO)

FOO %>% 
  filter(FOO > 0) %>% 
  plot_FOO()
```

![](FOO_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

### Roseate Tern chicks in 2018

``` r
# Filter the data and save to object
ROST_chicks_2018 <- df %>% 
  filter(Species == "ROST" & Age == "chick" & Year == 2018) %>% 
  select(River.herring:White.hake) 

# record number of records
n_samples <- nrow(ROST_chicks_2018)

FOO <- calc_FOO(ROST_chicks_2018)

FOO$Species <- scrub_periods(FOO)

FOO %>% 
  filter(FOO > 0) %>% 
  plot_FOO()
```

![](FOO_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

### Plotting them all together on one, multipanel figure

I think I can use faceting to do this, but I need to combine all the FOO
data together into one dataframe. First, save the FOO data for each
group.

``` r
FOO_COTE_chicks_2017 <- calc_FOO(COTE_chicks_2017)
FOO_COTE_chicks_2017$Year <- "2017"
FOO_COTE_chicks_2017$BirdSpecies <- "COTE"
FOO_COTE_chicks_2017$Age <- "chick"

FOO_COTE_chicks_2018 <- calc_FOO(COTE_chicks_2018)
FOO_COTE_chicks_2018$Year <- "2018"
FOO_COTE_chicks_2018$BirdSpecies <- "COTE"
FOO_COTE_chicks_2018$Age <- "chick"

FOO_ROST_chicks_2017 <- calc_FOO(ROST_chicks_2017)
FOO_ROST_chicks_2017$Year <- "2017"
FOO_ROST_chicks_2017$BirdSpecies <- "ROST"
FOO_ROST_chicks_2017$Age <- "chick"

FOO_ROST_chicks_2018 <- calc_FOO(ROST_chicks_2018)
FOO_ROST_chicks_2018$Year <- "2018"
FOO_ROST_chicks_2018$BirdSpecies <- "ROST"
FOO_ROST_chicks_2018$Age <- "chick"
```

Use row binding to add all the dataframes together so that I can use
faceting to plot.

``` r
All_FOO <- FOO_COTE_chicks_2017 %>% 
  bind_rows(., FOO_COTE_chicks_2018) %>% 
  bind_rows(., FOO_ROST_chicks_2017) %>% 
  bind_rows(., FOO_ROST_chicks_2018) 

All_FOO$Species <- scrub_periods(All_FOO)
```

I want to get rid of species that have zero FOO in all groups, so I can
do this by grouping by species and then filtering out species where FOO
= 0.

``` r
All_FOO %>% 
  group_by(Species) %>% 
  filter(FOO > 0) %>% 
  ggplot() +
  geom_bar(aes(x = Species, y = FOO, fill = Species), stat = "identity") +
  facet_grid(rows = vars(Year), cols = vars(BirdSpecies)) +
  theme_classic() +
  theme(axis.text.x = element_text(angle = 45,  hjust=1),
        panel.border=element_rect(colour="black",size=1, fill = NA)) +
  labs(y = "Frequency of occurrence (%)") +
  scale_fill_manual(values = Species_colours,
                    breaks = Species_ordered,
                    labels = Species_ordered) +
  theme(legend.position = "none") 
```

![](FOO_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->
