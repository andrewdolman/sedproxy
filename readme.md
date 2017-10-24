# Sedproxy: Simulation of Sediment Archived Climate Proxy Records.

------------------------------

## Introduction

`sedproxy` provides a forward model for sediment archived climate proxies. It is based on work described in Laepple and Huybers (2013). A manuscript is in preparation, Dolman and Laepple (in prep.), which will more fully describe the forward model and its applications. Please contact Dr Andrew Dolman <<andrew.dolman@awi.de>>, or Dr Thomas Laepple <<tlaepple@awi.de>>, at the Alfred-Wegener-Institute, Helmholtz Centre for Polar and Marine Research, Germany, for more information.

 
## Installation

**sedproxy** can be installed directly from bitbucket


```r
if (!require("devtools")) {
  install.packages("devtools")
}

devtools::install_bitbucket("ecus/sedproxy")
```

## Example data

`sedproxy` includes example data for a single sediment core and location: core number 41 in the Shakun et al. (2012) compilation (MD97-2141, Rosenthal et al. 2003). The climate signal is taken from the [TraCE-21ka](http://www.cgd.ucar.edu/ccr/TraCE/) Simulation of Transient Climate Evolution over the last 21,000 years, using the grid cell closest to core MD97-2141. Seasonality of *G.ruber*, the Foraminifera for which test Mg/Ca ratios were measured, is taken from the model of Fraile et al (2008). Sediment accumulation rates were estimated from the depth and age data associated with core MD97-2141, with a minimum rate of 0.2 * the mean rate.


**The MD97-2141 core**


```r
library(tidyverse)
library(knitr)
library(sedproxy)
```



```r
N41.proxy.details %>% 
  gather() %>% 
  kable(., format = "markdown")
```



|key             |value                                            |
|:---------------|:------------------------------------------------|
|Number          |41                                               |
|ID.no           |N41                                              |
|Core            |MD97-2141                                        |
|Location        |Sulu Sea                                         |
|Proxy           |Mg/Ca                                            |
|Lat             |8.78333                                          |
|Lon             |121.2833                                         |
|Elevation       |-3633.000000                                     |
|Reference       |Rosenthal et al., 2003                           |
|Resolution      |77.8894472361809                                 |
|Calibration.ref |Rosenthal and Lohman, 2002                       |
|Calibration     |T = ln(MgCa/0.28)/0.095                          |
|Foram.sp        |G. ruber                                         |
|Ref.14C         |de Garidel-Thoron et al., 2001, Paleoceanography |
|Notes           |NA                                               |
|Geo.cluster     |Sulu Sea                                         |
|Archive.type    |Marine sediment                                  |


**Input climate signal**

The first 5 rows:


```r
(N41.t21k.climate[1:5,]-273.15) %>% 
  kable(., format = "markdown", digits = 2)
```



|     1|     2|     3|     4|     5|     6|     7|     8|     9|    10|    11|    12|
|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|
| 24.71| 24.24| 24.91| 26.07| 26.53| 27.07| 27.40| 26.77| 26.49| 26.49| 26.78| 26.19|
| 24.84| 24.38| 24.68| 25.86| 26.57| 26.52| 27.01| 27.52| 26.63| 26.70| 26.63| 25.82|
| 24.69| 24.60| 25.21| 26.00| 26.46| 26.94| 27.00| 26.99| 26.39| 26.45| 26.66| 25.77|
| 24.55| 24.52| 25.34| 26.36| 26.87| 26.75| 27.29| 26.83| 26.55| 26.91| 26.59| 25.84|
| 24.62| 24.19| 24.80| 26.02| 26.84| 26.67| 26.99| 27.25| 26.80| 27.01| 26.67| 25.81|

**Actual proxy record**

Core MD97-2141 (Rosenthal et al. 2003)


```r
kable(head(N41.proxy), format = "markdown")
```



| Published.age| Published.temperature| Sed.acc.rate.m.yr|
|-------------:|---------------------:|-----------------:|
|      4334.286|                 28.92|         0.0003679|
|      4527.429|                 29.20|         0.0003675|
|      4575.714|                 29.15|         0.0003677|
|      4720.571|                 28.55|         0.0003677|
|      4913.714|                 28.33|         0.0003670|
|      4994.400|                 29.44|         0.0003667|

*******

## Function `ClimToProxyClim`

`ClimToProxyClim` is the main function in package `sedproxy`. It simulates a sediment archived proxy from an assumed true climate signal, the sediment accumulation rate, seasonality of the encoding organism/process, and the number of samples per timepoint.



```r
set.seed(26052017)
clim.in <- N41.t21k.climate[nrow(N41.t21k.climate):1,] - 273.15

PFM <- ClimToProxyClim(clim.signal = clim.in,
                       timepoints = round(N41.proxy$Published.age),
                       proxy.calibration.type = "identity",
                       seas.prod = N41.G.ruber.seasonality,
                       sed.acc.rate = N41.proxy$Sed.acc.rate.m.yr,
                       meas.noise = 0.46, n.samples = 30,
                       n.replicates = 10)
```



```r
PFM$everything
```

```
## # A tibble: 8,213 x 4
##    timepoints replicate              stage    value
##         <dbl>     <dbl>              <chr>    <dbl>
##  1       4334         1 proxy.bt.sb.sampYM 28.52609
##  2       4527         1 proxy.bt.sb.sampYM 27.48838
##  3       4576         1 proxy.bt.sb.sampYM 28.02794
##  4       4721         1 proxy.bt.sb.sampYM 27.19180
##  5       4914         1 proxy.bt.sb.sampYM 27.38228
##  6       4994         1 proxy.bt.sb.sampYM 27.85321
##  7       5092         1 proxy.bt.sb.sampYM 27.87501
##  8       5156         1 proxy.bt.sb.sampYM 27.83270
##  9       5254         1 proxy.bt.sb.sampYM 27.46396
## 10       5318         1 proxy.bt.sb.sampYM 27.36841
## # ... with 8,203 more rows
```

**Simple plotting**


```r
PFM$everything %>% 
  PlotPFMs(max.replicates = 1)
```

![](readme_files/figure-html/default_plot-1.png)<!-- -->


**Plot 5 replicates of the final simulated proxy**


```r
PFM$everything %>% 
  filter(stage == "simulated.proxy") %>% 
  PlotPFMs(., max.replicates = 5)
```

![](readme_files/figure-html/plot_reps-1.png)<!-- -->















## Literature cited

Fraile, I., Schulz, M., Mulitza, S., & Kucera, M. (2008): Predicting the global distribution of planktonic foraminifera using a dynamic ecosystem model. Biogeosciences, 5: 891–911.

Laepple, T., & Huybers, P. (2013): Reconciling discrepancies between Uk37 and Mg/Ca reconstructions of Holocene marine temperature variability. Earth and Planetary Science Letters, 375: 418–429.

Rosenthal, Y., Oppo, D. W., & Linsley, B. K. (2003): The amplitude and phasing of climate change during the last deglaciation in the Sulu Sea, western equatorial Pacific. Geophys. Res. Lett., 30: 1428.

Shakun, J. D., Clark, P. U., He, F., Marcott, S. A., Mix, A. C., Liu, Z., Otto-Bliesner, B., Schmittner, A., & Bard, E. (2012): Global warming preceded by increasing carbon dioxide concentrations during the last deglaciation. Nature, 484: 49–54.


