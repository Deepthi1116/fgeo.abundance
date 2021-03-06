Basal area: Comparing output from Suzanne and mine
================

Suzanne wrote:

> Most of the numbers coincide. However, for some species
> (i.e. buttressed trees), they do not: Ceiba pentandra, Alseis
> blackiana. Dipteryx oleifera, etc. The difference in census 3 is quite
> high. Can you please check to see why you got such a high number for
> this census? The numbers after this census are also quite high.

> Remember that you have to make a table with one record per census per
> stemid. The stems from buttressed trees may be measured more than once
> in a census but at different heights. So you have to choose the record
> where the HOM is hightest for any stemid. I think that you just summed
> all the records without taking this into consideration.

Thanks Suzanne. I had a bug here: When I changed the code to sum all
stems in each tree (within the given DBH range) I unintentionally
disabled the code that picks the (single) StemID with largest HOM. Now
that’s fixed and added a formal test so this doesn’t happen again. Now
the basal area of buttressed species should be lower than before.

``` r
library(tidyverse)
library(fgeo)
```

Outputs by me and Suzanne

``` r
path_mao <- here::here(
  "inst/issues/59_abund_tables/out/bci_sapling_tree_basal_area.csv"
)
mao <- readr::read_csv(path_mao)

path_lao <- here::here("inst/issues/59_abund_tables/lao/BasalArea.xlsx")
lao <- readxl::read_excel(path_lao)
lao <- unite(lao, "species", Genus, SpeciesName, sep = " ")
```

Some species that have buttresses.

``` r
spp <- c("Ceiba pentandra", "Alseis blackiana", "Dipteryx oleifera")
spp
#> [1] "Ceiba pentandra"   "Alseis blackiana"  "Dipteryx oleifera"
```

Helper funciton to reduce duplication.

``` r
plot_spp <- function(x) {
  x %>% 
    filter(species %in% spp) %>% 
  ggplot(aes(census, ba)) +
  geom_col(aes(fill = species), position = "dodge")
}
```

Figure from data by Suzanne.

``` r
lao %>% 
  mutate(census = plotcensusnumber, ba = ba_over_50) %>% 
  plot_spp() +
  labs(title = "Suzanne's data")
```

<img src="71_high_ba_files/figure-gfm/unnamed-chunk-3-1.png" width="70%" style="display: block; margin: auto;" />

Compare with figure from data calculated with `basal_area_byyr()` after
fixing bug reported in issue 71
(<https://github.com/forestgeo/fgeo.abundance/issues/71>). Now it looks
very similar to Suzanne’s data.

``` r
mao %>% 
  filter(species %in% spp) %>% 
  gather("years", "ba", `1982`:`2015`) %>% 
  mutate(census = as.integer(as.factor(years))) %>% 
  ggplot(aes(census, ba)) +
  geom_col(aes(fill = species), position = "dodge") +
  labs(title = "Mauro's data")
```

<img src="71_high_ba_files/figure-gfm/unnamed-chunk-4-1.png" width="70%" style="display: block; margin: auto;" />
