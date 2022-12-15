---
layout: default
title: Inspection
parent: mvpaShiny
nav_order: 4
---

## Inspection tab

The inspection tab offers users the options to create quantile-quantile-plots and visualize variable correlation pattern.



### Quantile-Quantile plots

![inspection_qq_plot](assets/images/Inspection/inspection_qq_plot.png)

The purpose of quantile-quantile plots (QQ-plots) is to check whether a variable shows a normal distribution. If a variable is perfectly normally distributed each data point will follow the faint black diagonal which represents a theoretical normal distribution for the given data.

*mvpaShiny* allows to show several QQ-plots side-by-side, using the multiple selection field.



### Correlation plots

![inspection_correlation_single_response](assets/images/Inspection/inspection_correlation_single_response.png)

*mvpaShiny* can plot two types of correlation:

- Variables with (a single) response. 
  The plot can be rotated by 90° for easier interpretation.
- Variables with each other



![inspection_correlation_matrix](assets/images/Inspection/inspection_correlation_matrix.png)
