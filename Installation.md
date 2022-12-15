---
layout: default
title: Installation
nav_order: 2
---

## Installation guide

To install the local version of *mvpaShiny* please have **R Studio** (version ≥ 2022.02.3, Build 492) and **R** (version ≥ 4.1.X) installed.

[Download R Studio](https://www.rstudio.com/products/rstudio/download/)

[Download R](https://cran.r-project.org/bin/windows/base/)



### Code to install and run *mvpaShiny*

Please copy-paste the code below in the R Studio console and execute it.

To install the package the *devtools* package is required which the following code will try to install automatically if it has not been installed yet.

```R
if (!require("devtools", quietly = TRUE)) {
  install.packages("devtools") 
} else {
  print("Devtools has been already installed.")
}
    
devtools::install_github("tim-b90/mvpaShiny")
```

Start *mvpaShiny*!

```R
library(mvpaShiny)
mvpaApp()
```





