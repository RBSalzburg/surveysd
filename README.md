
[![Build Status](https://travis-ci.org/statistikat/surveysd.svg?branch=master)](https://travis-ci.org/statistikat/surveysd) <!--[![Coverage Status](https://coveralls.io/repos/github/statistikat/surveysd/badge.svg?branch=master)](https://coveralls.io/github/statistikat/surveysd?branch=master)--> <!--[![CRAN](http://www.r-pkg.org/badges/version/surveysd)](https://CRAN.R-project.org/package=surveysd)--> <!--[![Downloads](http://cranlogs.r-pkg.org/badges/surveysd)](https://CRAN.R-project.org/package=surveysd)--> <!--[![Mentioned in Awesome Official Statistics ](https://awesome.re/mentioned-badge.svg)](http://www.awesomeofficialstatistics.org)-->

surveysd
--------

This is the development place for R-package `surveysd`

Estimate standard deviation of estimates in complex surveys using bootstrap weights.

### Installation

This package can be installed like any other R package on github via `devtools::install_github`

``` r
library(devtools)
install_github("statistikat/surveysd")
```

------------------------------------------------------------------------

### Syntax example

#### Load dummy data

``` r
library(surveysd)
eusilc <- surveysd:::demo.eusilc(n = 2, prettyNames = TRUE)
```

#### Draw bootstrap replicates

Use stratified resampling without replacement to generate 10 samples. Those samples are consistent with respect to the reference periods.

``` r
dat_boot <- draw.bootstrap(eusilc, REP = 10, hid = "hid", weights = "pWeight", 
                           strata = "region", period = "year")
```

#### Calibrate bootstrap replicates

Calibrate each sample according to the distribution of `gender` (on a personal level) and `region` (on a household level).

``` r
dat_boot_calib <- recalib(dat_boot, conP.var = "gender", conH.var = "region")
```

    ## Convergence reached in  3  steps 
    ## Convergence reached in  1  steps 
    ## Convergence reached in  2  steps 
    ## Convergence reached in  3  steps 
    ## Convergence reached in  2  steps 
    ## Convergence reached in  3  steps 
    ## Convergence reached in  2  steps 
    ## Convergence reached in  2  steps 
    ## Convergence reached in  1  steps 
    ## Convergence reached in  3  steps

#### Estimate with respect to a grouping variable

Estimate relative amount of persons at risk of poverty per period and `gender`.

``` r
err.est <- calc.stError(dat_boot_calib, var = "povertyRisk", fun = weightedRatio,
                        group = "gender", period.diff = NULL, period.mean = NULL)
err.est$Estimates
```

    ##    year     n       N gender val_povertyRisk stE_povertyRisk
    ## 1: 2010  7267 3979572   male        12.02660       0.4171625
    ## 2: 2010  7560 4202650 female        16.73351       0.3793100
    ## 3: 2010 14827 8182222   <NA>        14.44422       0.3772259
    ## 4: 2011  7267 3979572   male        12.63771       0.3255330
    ## 5: 2011  7560 4202650 female        16.08442       0.3462576
    ## 6: 2011 14827 8182222   <NA>        14.40805       0.2861940

The output contains estimates (`val_povertyRisk`) as well as standard errors (`stE_povertyRisk`) measured in percent.

#### Estimate with respect to several variables

Estimate relative amount of persons at risk of poverty per period for each `region`, `gender`, and combination of both.

``` r
group <- list("gender", "region", c("gender", "region"))
err.est <- calc.stError(dat_boot_calib, var = "povertyRisk", fun = weightedRatio,
                        group = group, period.diff = NULL, period.mean = NULL)
head(err.est$Estimates)
```

    ##    year   n        N gender     region val_povertyRisk stE_povertyRisk
    ## 1: 2010 261 122741.8   male Burgenland       17.414524        2.236535
    ## 2: 2010 288 137822.2 female Burgenland       21.432598        2.750966
    ## 3: 2010 359 182732.9   male Vorarlberg       12.973259        3.789209
    ## 4: 2010 374 194622.1 female Vorarlberg       19.883637        2.244750
    ## 5: 2010 440 253143.7   male   Salzburg        9.156964        1.727869
    ## 6: 2010 484 282307.3 female   Salzburg       17.939382        1.414657

``` r
## skipping 54 more rows
```
