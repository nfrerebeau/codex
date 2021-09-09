
<!-- README.md is generated from README.Rmd. Please edit that file -->

# arkhe <img width=120px src="man/figures/logo.png" align="right" />

<!-- badges: start -->

[![R-CMD-check](https://github.com/tesselle/arkhe/workflows/R-CMD-check/badge.svg)](https://github.com/tesselle/arkhe/actions)
[![codecov](https://codecov.io/gh/tesselle/arkhe/branch/master/graph/badge.svg)](https://codecov.io/gh/tesselle/arkhe)

[![r-universe](https://tesselle.r-universe.dev/badges/arkhe)](https://tesselle.r-universe.dev)
[![CRAN
Version](http://www.r-pkg.org/badges/version/arkhe)](https://cran.r-project.org/package=arkhe)
[![CRAN
checks](https://cranchecks.info/badges/worst/arkhe)](https://cran.r-project.org/web/checks/check_results_arkhe.html)
[![CRAN
Downloads](http://cranlogs.r-pkg.org/badges/arkhe)](https://cran.r-project.org/package=arkhe)

[![Project Status: Active – The project has reached a stable, usable
state and is being actively
developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.3526659.svg)](https://doi.org/10.5281/zenodo.3526659)
<!-- badges: end -->

## Overview

A collection of classes that represent archaeological data. This package
provides a set of S4 classes that represent different special types of
matrix (absolute/relative frequency, presence/absence data,
co-occurrence matrix, etc.) upon which package developers can build
subclasses. It also provides a set of generic methods (mutators and
coercion mechanisms) and functions (e.g. predicates). In addition, a few
classes of general interest (e.g. that represent stratigraphic
relationships) are implemented.

## Installation

You can install the released version of **arkhe** from
[CRAN](https://CRAN.R-project.org) with:

``` r
install.packages("arkhe")
```

And the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("remotes")
remotes::install_github("tesselle/arkhe")
```

## Usage

``` r
## Load the package
library(arkhe)
```

**arkhe** provides a set of S4 classes that represent different special
types of matrix.

-   Integer matrix:
    -   `CountMatrix` represents absolute frequency data,
    -   `OccurrenceMatrix` represents a co-occurrence matrix,
-   Numeric matrix:
    -   `CompositionMatrix` represents relative frequency
        (compositional) data,
-   Logical matrix:
    -   `IncidenceMatrix` represents presence/absence data,
    -   `StratigraphicMatrix` represents stratigraphic relationships.

*It assumes that you keep your data tidy*: each variable (type/taxa)
must be saved in its own column and each observation (assemblage/sample)
must be saved in its own row.

These new classes are of simple use as they inherit from the base
`matrix`:

``` r
## Define a count data matrix
## (data will be rounded to zero decimal places, then coerced with as.integer)
quanti <- CountMatrix(data = sample(0:10, 100, TRUE), nrow = 10, ncol = 10)

## Define a logical matrix
## (data will be coerced with as.logical)
quali <- IncidenceMatrix(data = sample(0:1, 100, TRUE), nrow = 10, ncol = 10)
```

**arkhe** uses coercing mechanisms (with validation methods) for data
type conversions:

``` r
## Create a count matrix
X <- matrix(data = sample(0:10, 75, TRUE), nrow = 15, ncol = 5)

## Coerce to absolute frequencies
A1 <- as_count(X)

## Coerce to relative frequencies
B <- as_composition(A1)

## Row sums are internally stored before coercing to a frequency matrix
## (use get_totals() to get these values)
## This allows to restore the source data
A2 <- as_count(B)
all(A1 == A2)
#> [1] TRUE

## Coerce to presence/absence
D <- as_incidence(A1)

## Coerce to a co-occurrence matrix
E <- as_occurrence(A1)
```

The `CountMatrix`, `CompositionMatrix` and `IncidenceMatrix` classes
have special slots:

-   `samples` for replicated measurements/observation,
-   `groups` to group data by site/area,
-   `tqp` and `taq` to specify the chronology of an assemblage.

The way chronological information is handled is somewhat opinionated:

-   Sub-annual precision is overkill/meaningless in most situations:
    dates are assumed to be expressed in years CE and are stored as
    integers (values are coerced with `as.integer()` and hence truncated
    towards zero).
-   A date range
    (i.e. <abbr title="terminus post quem">TPQ</abbr>/<abbr title="terminus ante quem">TAQ</abbr>)
    is expected, although it is possible to specify a point estimate.

When coercing a `data.frame` to a `*Matrix` object, an attempt is made
to automatically assign values to these slots by mapping column names.
This behavior can be disabled by setting
`options(arkhe.autodetect = FALSE)`.

``` r
Y <- as.data.frame(X)
Y$samples <- rep(c("a", "b", "c", "d", "e"), each = 3)
Y$groups <- rep(c("A", "B", "C"), each = 5)
Y$tpq <- sample(1301:1400, 15, TRUE) # TPQ
Y$taq <- sample(1451:1500, 15, TRUE) # TAQ

## Coerce to a count matrix
Z <- as_count(Y)

## Get groups
get_samples(Z)
#>  [1] "a" "a" "a" "b" "b" "b" "c" "c" "c" "d" "d" "d" "e" "e" "e"

## Get groups
get_groups(Z)
#>  [1] "A" "A" "A" "A" "A" "B" "B" "B" "B" "B" "C" "C" "C" "C" "C"

## Get dates
get_dates(Z)
#>        tpq  taq
#> row1  1350 1456
#> row2  1302 1451
#> row3  1357 1483
#> row4  1398 1475
#> row5  1345 1485
#> row6  1397 1484
#> row7  1303 1471
#> row8  1340 1462
#> row9  1394 1476
#> row10 1304 1478
#> row11 1363 1466
#> row12 1318 1478
#> row13 1334 1471
#> row14 1362 1491
#> row15 1400 1491
```

## Contributing

Please note that the **arkhe** project is released with a [Contributor
Code of Conduct](https://www.tesselle.org/conduct.html). By contributing
to this project, you agree to abide by its terms.
