Dplyr vs Pandas
================
cwpkuo
2022-08-12

-   <a href="#setup" id="toc-setup">Setup</a>
-   <a href="#exploration" id="toc-exploration">Exploration</a>
-   <a href="#manipulate-dimensions"
    id="toc-manipulate-dimensions">Manipulate Dimensions</a>
-   <a href="#group-and-summarize" id="toc-group-and-summarize">Group and
    Summarize</a>
-   <a href="#joins" id="toc-joins">Joins</a>

## Setup

Use Starwars dataframe from R (dplyr) and load in Python as Pandas
DataFrame

``` r
# R
if (!require("pacman")) install.packages("pacman")
if (!require("dplyr")) install.packages("dplyr")
pacman::p_load(dplyr)

df <- starwars %>% mutate(height = coalesce(height, 0)) # cleanup NAs

df_sex <- data.frame(
  sex = c("male", "female"),
  sex_cd = c("m", "f")
) # lookup table for joins
```

``` python
# Python
import pandas as pd
import numpy as np

df = pd.DataFrame(r.df)

df_sex = pd.DataFrame(r.df_sex)
```

## Exploration

### Table Dimensions

``` r
# R
dim(df)
```

    ## [1] 87 14

``` python
# Python
df.shape
```

    ## (87, 14)

### Head

``` r
# R
df %>% head(2)
```

    ## # A tibble: 2 x 14
    ##   name      height  mass hair_color skin_color eye_color birth_year sex   gender
    ##   <chr>      <dbl> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
    ## 1 Luke Sky~    172    77 blond      fair       blue              19 male  mascu~
    ## 2 C-3PO        167    75 <NA>       gold       yellow           112 none  mascu~
    ## # ... with 5 more variables: homeworld <chr>, species <chr>, films <list>,
    ## #   vehicles <list>, starships <list>

``` python
# Pandas
df.head(2)
```

    ##              name  ...                   starships
    ## 0  Luke Skywalker  ...  [X-wing, Imperial shuttle]
    ## 1           C-3PO  ...                          []
    ## 
    ## [2 rows x 14 columns]

### Summary Statistics

``` r
# R
summary(df$height)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##     0.0   164.0   178.0   162.3   190.5   264.0

``` python
# Pandas
df['height'].describe()
```

    ## count     87.000000
    ## mean     162.333333
    ## std       55.671726
    ## min        0.000000
    ## 25%      164.000000
    ## 50%      178.000000
    ## 75%      190.500000
    ## max      264.000000
    ## Name: height, dtype: float64

## Manipulate Dimensions

### Filtering

``` r
# R
df %>% filter(species %in% c("Human", "Wookiee") & height >= 200) %>% select(name, species, height)
```

    ## # A tibble: 3 x 3
    ##   name        species height
    ##   <chr>       <chr>    <dbl>
    ## 1 Darth Vader Human      202
    ## 2 Chewbacca   Wookiee    228
    ## 3 Tarfful     Wookiee    234

``` python
# Pandas
df.query('species in ("Human", "Wookiee") & height >= 200').filter(['name', 'species', 'height'])
```

    ##            name  species  height
    ## 3   Darth Vader    Human   202.0
    ## 12    Chewbacca  Wookiee   228.0
    ## 77      Tarfful  Wookiee   234.0

``` python
df.loc[df['species'].isin(['Human', 'Wookiee']) & (df['height'] >= 200), ['name', 'species', 'height']]
```

    ##            name  species  height
    ## 3   Darth Vader    Human   202.0
    ## 12    Chewbacca  Wookiee   228.0
    ## 77      Tarfful  Wookiee   234.0

### Subset Columns

``` r
# R
df %>% slice(1:2) %>% select(starts_with("na"))
```

    ## # A tibble: 2 x 1
    ##   name          
    ##   <chr>         
    ## 1 Luke Skywalker
    ## 2 C-3PO

``` python
# Pandas
df.iloc[0:2].filter(regex = '^na')
```

    ##              name
    ## 0  Luke Skywalker
    ## 1           C-3PO

### Drop Columns

``` r
# R
df %>% head(1) %>% select(-(height:starships))
```

    ## # A tibble: 1 x 1
    ##   name          
    ##   <chr>         
    ## 1 Luke Skywalker

``` python
# Pandas
df.head(1).drop(columns = ['height', 'mass', 'hair_color', 'skin_color', 'eye_color', 'birth_year', 'sex', 'gender', 'homeworld', 'species', 'films', 'vehicles', 'starships'])
```

    ##              name
    ## 0  Luke Skywalker

``` python
df.head(1).drop(df.loc[:, 'height':'starships'], axis = 1)
```

    ##              name
    ## 0  Luke Skywalker

### Drop Duplicates & NAs

``` r
# R
df %>% select(gender) %>% distinct() %>% filter(!is.na(gender))
```

    ## # A tibble: 2 x 1
    ##   gender   
    ##   <chr>    
    ## 1 masculine
    ## 2 feminine

``` python
# Pandas
df.filter(['gender']).drop_duplicates().dropna().query('gender != "NA"')
```

    ##       gender
    ## 0  masculine
    ## 4   feminine

### Sampling

``` r
# R
df %>% sample_n(2)
```

    ## # A tibble: 2 x 14
    ##   name      height  mass hair_color skin_color eye_color birth_year sex   gender
    ##   <chr>      <dbl> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
    ## 1 Ayla Sec~    178    55 none       blue       hazel             48 fema~ femin~
    ## 2 Tion Med~    206    80 none       grey       black             NA male  mascu~
    ## # ... with 5 more variables: homeworld <chr>, species <chr>, films <list>,
    ## #   vehicles <list>, starships <list>

``` r
df %>% sample_frac(0.02)
```

    ## # A tibble: 2 x 14
    ##   name      height  mass hair_color skin_color eye_color birth_year sex   gender
    ##   <chr>      <dbl> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
    ## 1 Poggle t~    183    80 none       green      yellow            NA male  mascu~
    ## 2 Sly Moore    178    48 none       pale       white             NA <NA>  <NA>  
    ## # ... with 5 more variables: homeworld <chr>, species <chr>, films <list>,
    ## #   vehicles <list>, starships <list>

``` python
# Pandas
df.sample(n = 2)
```

    ##                  name  height  ...  vehicles            starships
    ## 59  Poggle the Lesser   183.0  ...        []                   []
    ## 83        Poe Dameron     0.0  ...        []  T-70 X-wing fighter
    ## 
    ## [2 rows x 14 columns]

``` python
df.sample(frac = 0.02)
```

    ##                  name  ...                                          starships
    ## 10   Anakin Skywalker  ...  [Trade Federation cruiser, Jedi Interceptor, N...
    ## 8   Biggs Darklighter  ...                                             X-wing
    ## 
    ## [2 rows x 14 columns]

### Slicing

``` r
# R
df %>% slice_max(height, n = 2)
```

    ## # A tibble: 2 x 14
    ##   name      height  mass hair_color skin_color eye_color birth_year sex   gender
    ##   <chr>      <dbl> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
    ## 1 Yarael P~    264    NA none       white      yellow            NA male  mascu~
    ## 2 Tarfful      234   136 brown      brown      blue              NA male  mascu~
    ## # ... with 5 more variables: homeworld <chr>, species <chr>, films <list>,
    ## #   vehicles <list>, starships <list>

``` python
# Pandas
df.nlargest(2, 'height')
```

    ##            name  height   mass  ...                films vehicles starships
    ## 53  Yarael Poof   264.0    NaN  ...   The Phantom Menace       []        []
    ## 77      Tarfful   234.0  136.0  ...  Revenge of the Sith       []        []
    ## 
    ## [2 rows x 14 columns]

### Sorting

``` r
# R
df %>% filter(!is.na(mass)) %>% arrange(desc(sex), desc(mass)) %>% select(sex, mass)
```

    ## # A tibble: 59 x 2
    ##    sex    mass
    ##    <chr> <dbl>
    ##  1 none    140
    ##  2 none     75
    ##  3 none     32
    ##  4 none     32
    ##  5 male    159
    ##  6 male    136
    ##  7 male    136
    ##  8 male    120
    ##  9 male    113
    ## 10 male    112
    ## # ... with 49 more rows

``` python
# Pandas
df.query('mass.notna()').sort_values(['sex', 'mass'], ascending = False).filter(['sex', 'mass']).head(10)
```

    ##      sex   mass
    ## 21  none  140.0
    ## 1   none   75.0
    ## 2   none   32.0
    ## 7   none   32.0
    ## 76  male  159.0
    ## 3   male  136.0
    ## 77  male  136.0
    ## 5   male  120.0
    ## 22  male  113.0
    ## 12  male  112.0

### Renaming Column

``` r
# R
df %>% rename(hw = homeworld) %>% select(hw)
```

    ## # A tibble: 87 x 1
    ##    hw      
    ##    <chr>   
    ##  1 Tatooine
    ##  2 Tatooine
    ##  3 Naboo   
    ##  4 Tatooine
    ##  5 Alderaan
    ##  6 Tatooine
    ##  7 Tatooine
    ##  8 Tatooine
    ##  9 Tatooine
    ## 10 Stewjon 
    ## # ... with 77 more rows

``` python
# Pandas
df.rename(columns = {'homeworld': 'hw'}).filter(['hw'])
```

    ##           hw
    ## 0   Tatooine
    ## 1   Tatooine
    ## 2      Naboo
    ## 3   Tatooine
    ## 4   Alderaan
    ## ..       ...
    ## 82        NA
    ## 83        NA
    ## 84        NA
    ## 85        NA
    ## 86     Naboo
    ## 
    ## [87 rows x 1 columns]

### Creating New Column

``` r
# R
df %>% mutate(eye_color_bin = case_when(eye_color %in% c('black', 'brown', 'dark') ~ 'dark',
                                        eye_color %in% c('unknown') ~ 'unknown',
                                        TRUE ~ 'bright')) %>%
  group_by(eye_color_bin) %>% 
  summarise(n()) %>% 
  ungroup()
```

    ## # A tibble: 3 x 2
    ##   eye_color_bin `n()`
    ##   <chr>         <int>
    ## 1 bright           52
    ## 2 dark             32
    ## 3 unknown           3

``` python
# Pandas/Numpy
df['eye_color_bin'] = np.where(df['eye_color'].isin(['black', 'brown', 'dark']), 'dark', 
                      np.where(df['eye_color'].isin(['unknown']), 'unknown', 'bright'))
                      
df.groupby(['eye_color_bin']).size()
```

    ## eye_color_bin
    ## bright     52
    ## dark       32
    ## unknown     3
    ## dtype: int64

``` r
# R
df %>% mutate(height_2x = height*2,
              height_4x = height*4) %>% 
  select(height, height_2x, height_4x) %>% 
  head(1)
```

    ## # A tibble: 1 x 3
    ##   height height_2x height_4x
    ##    <dbl>     <dbl>     <dbl>
    ## 1    172       344       688

``` python
# Pandas
df.assign(height_2x = lambda x: x['height']*2, 
          height_4x = lambda x: x['height_2x']*2).filter(['height', 'height_2x', 'height_4x']).head(1)
```

    ##    height  height_2x  height_4x
    ## 0   172.0      344.0      688.0

## Group and Summarize

### Records by Group

``` r
# R
df %>% group_by(sex) %>% summarise(count = n())
```

    ## # A tibble: 5 x 2
    ##   sex            count
    ##   <chr>          <int>
    ## 1 female            16
    ## 2 hermaphroditic     1
    ## 3 male              60
    ## 4 none               6
    ## 5 <NA>               4

``` python
# Pandas
df.groupby(['sex']).size()
```

    ## sex
    ## NA                 4
    ## female            16
    ## hermaphroditic     1
    ## male              60
    ## none               6
    ## dtype: int64

### Aggregations by Group

``` r
# R
df %>% group_by(sex) %>% summarise(height_mean = mean(height, na.rm = T), 
                                   height_max  = max(height, na.rm = T),
                                   height_diff = max(height, na.rm = T) - min(height, na.rm = T))
```

    ## # A tibble: 5 x 4
    ##   sex            height_mean height_max height_diff
    ##   <chr>                <dbl>      <dbl>       <dbl>
    ## 1 female                159.        213         213
    ## 2 hermaphroditic        175         175           0
    ## 3 male                  170.        264         264
    ## 4 none                  109.        200         200
    ## 5 <NA>                  136         183         183

``` python
# Pandas
df.groupby(['sex'], as_index = False).agg(
  {'height': ['mean', 'min', 'max'],
  'homeworld': 'first'}
  )
```

    ##               sex      height                homeworld
    ##                          mean    min    max      first
    ## 0              NA  136.000000    0.0  183.0      Naboo
    ## 1          female  158.687500    0.0  213.0   Alderaan
    ## 2  hermaphroditic  175.000000  175.0  175.0  Nal Hutta
    ## 3            male  170.150000    0.0  264.0   Tatooine
    ## 4            none  109.333333    0.0  200.0   Tatooine

``` python
# Pandas
df.groupby(['sex'], as_index = False).agg(
  height_mean = pd.NamedAgg('height', 'mean'),
  height_max =  pd.NamedAgg('height', 'max'),
  height_diff = pd.NamedAgg('height', lambda x: max(x) - min(x))
  )
```

    ##               sex  height_mean  height_max  height_diff
    ## 0              NA   136.000000       183.0        183.0
    ## 1          female   158.687500       213.0        213.0
    ## 2  hermaphroditic   175.000000       175.0          0.0
    ## 3            male   170.150000       264.0        264.0
    ## 4            none   109.333333       200.0        200.0

## Joins

### Left Join

``` r
# R
df %>% left_join(df_sex, by = c("sex" = "sex")) %>% select(sex, sex_cd) %>% head(1)
```

    ## # A tibble: 1 x 2
    ##   sex   sex_cd
    ##   <chr> <chr> 
    ## 1 male  m

``` r
# Can use inner_join(), right_join() and 
```

``` python
# Pandas
df.merge(df_sex, how = 'left', left_on = 'sex', right_on = 'sex').filter(['sex', 'sex_cd']).head(1)

# how = 'inner', 'right', 'outer'
```

    ##     sex sex_cd
    ## 0  male      m

### Anti Join

``` r
# R
df %>% anti_join(df_sex, by = c("sex" = "sex")) %>% select(sex) %>% head(1)
```

    ## # A tibble: 1 x 1
    ##   sex  
    ##   <chr>
    ## 1 none

``` python
# Pandas
df.merge(df_sex, how = 'outer', left_on = 'sex', right_on = 'sex', indicator = True).query('_merge in ["left_only"]').filter(['sex']).head(1)
```

    ##      sex
    ## 60  none
