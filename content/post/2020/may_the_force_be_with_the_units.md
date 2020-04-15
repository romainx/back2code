---
title: 'May the force be with the units'
date: '2020-04-15'
categories: [data]
tags: ['R']
---

# Introduction

When working with data it's very important to be aware of the unit of each variable. To make it **explicit** it's convenient to use the package called [`units`](https://github.com/r-quantities/units/).

> Support for measurement units in R vectors, matrices and arrays: automatic propagation, conversion, derivation and simplification of units; raising errors in case of unit incompatibility. 

# A simple example

By reading the help of the `starwars` dataset, we learn that

- `height`: Height (cm)
- `mass`: Weight (kg)

So let's test it.

```R
library(dplyr)
library(magrittr)
library(units)

starwars %>% 
    mutate(height = set_units(height, cm)) %>%
    mutate(mass = set_units(mass, kg)) %>% 
    select(name, height, mass, species) %>% 
    head()

# A tibble: 6 x 4
#   name           height  mass species
#   <chr>            [cm]  [kg] <chr>  
# 1 Luke Skywalker    172    77 Human  
# 2 C-3PO             167    75 Droid  
# 3 R2-D2              96    32 Droid  
# 4 Darth Vader       202   136 Human  
# 5 Leia Organa       150    49 Human  
# 6 Owen Lars         178   120 Human 
```

Note that units are nicely printed in the output.

# Converting units

Imagine I want to draw a bar chart of the 5 first characters of the dataset. But I want the **scale to be defined in meters** instead of centimeters since it make it cleared to figure out the scale.
However, it does not work as is. To make it work the package [`ggforce`](https://ggforce.data-imaginist.com/) is needed and more specifically, the [`scale_unit`](https://ggforce.data-imaginist.com/reference/scale_unit.html) function.

```R
library(ggforce)
starwars %>% 
    mutate(height = set_units(height, cm)) %>%
    mutate(mass = set_units(mass, kg)) %>% 
    select(name, height, mass, species) %>% 
    head(5) %>%
    ggplot() + 
    geom_bar(aes(x = reorder(name, -height), y = height, 
                 colour = species, 
                 fill = after_scale(alpha(colour, 0.5))), 
                 stat = "identity", size= 1.5) + # Give a good looking
    xlab("name") +
    scale_y_unit(unit = "m") + # Doing the conversion
    scale_x_discrete(guide = guide_axis(n.dodge = 2)) # To avoid overlapp in names
```

![Size of the characters](/post/may_the_force_be_with_the_units/starwars_height.png)

Thanks to a call to `scale_y_unit(unit = "m")` I'm able to display the scale in meters.

*Note: `x = reorder(name, -height)` is a trick to sort the bars by value (height in this case).*

# Mixing units

If we try to check if **height is correlated to weight** we obtain this scatter plot.

```R
starwars %>% 
    mutate(height = set_units(height, cm)) %>%
    mutate(mass = set_units(mass, kg)) %>% 
    select(name, height, mass, species) %>% 
    drop_na() %>%
    ggplot() + 
    geom_point(aes(x = mass, y = height))
```

![Mass vs Height with outliers](/post/may_the_force_be_with_the_units/starwars_outliers.png)

There is a *big* outlier in this plot. Let's check who is it?

```R
starwars %>% 
    mutate(height = set_units(height, cm)) %>%
    mutate(mass = set_units(mass, kg)) %>% 
    select(name, height, mass, species) %>% 
    drop_na() %>%
    filter(mass == max(mass))

# A tibble: 1 x 4
# name                  height  mass species
# <chr>                   [cm]  [kg] <chr>  
# 1 Jabba Desilijic Tiure    175  1358 Hutt  
```

It's **Jabba the Hutt**. For him it would be also useful to use tons instead of kg!
If we remove him we obtain something more conventional.

```R
starwars %>% 
    mutate(height = set_units(height, cm)) %>%
    mutate(mass = set_units(mass, kg)) %>% 
    select(name, height, mass, species) %>% 
    drop_na() %>%
    filter(mass != max(mass)) %>% # Here is the removal
    ggplot() + 
    geom_point(aes(x = mass, y = height))
```

![Mass vs Height without outliers](/post/may_the_force_be_with_the_units/starwars_without_outliers.png)

His Body Mass Index (BMI)[^1] should be very bad let's check it. Thanks to the `units` package it's **pretty straightforward and explicit**.

```R
starwars %>% 
    mutate(height = set_units(height, cm)) %>%
    mutate(mass = set_units(mass, kg)) %>%
    # Converting in meters and raising to power of 2
    mutate(bmi = mass / set_units(height, m) ^ 2) %>%
    select(name, height, mass, bmi, species) %>%
    arrange(desc(bmi)) %>%
    head()

# A tibble: 6 x 5
#   name                  height  mass       bmi species       
#   <chr>                   [cm]  [kg]  [kg/m^2] <chr>         
# 1 Jabba Desilijic Tiure    175  1358 443.42857 Hutt          
# 2 Dud Bolt                  94    45  50.92802 Vulptereen    
# 3 Yoda                      66    17  39.02663 Yoda's species
# 4 Owen Lars                178   120  37.87401 Human         
# 5 IG-88                    200   140  35.00000 Droid         
# 6 R2-D2                     96    32  34.72222 Droid    
```

Note that the BMI `bmi` unit is correct: `kg/m^2`.

# References

- [Is it possible to use ggplot with the units package in R?](https://stackoverflow.com/questions/61209769/is-it-possible-to-use-ggplot-with-the-units-package-in-r/)
- [Measurement units in R](https://cran.r-project.org/web/packages/units/vignettes/measurement_units_in_R.html)

[^1]: The BMI is defined as the body mass divided by the square of the body height, and is universally expressed in units of kg/m2, resulting from mass in kilograms and height in metres [Wikipedia](https://en.wikipedia.org/wiki/Body_mass_index).