---
layout: default
title: Covariate projection
parent: mvpaShiny
nav_order: 6
---

## Covariate projection

Covariates  (or confounders) are variables that are associated with both an explanatory and a response variable. In other words, they interfere with or obscure the relationship between the explanatory and response variable.  An  example is the influence of adiposity (covariate) on the relationship of physical activity (independent variable) and the resulting aerobic fitness (response).

Covariance projection is a method to adjust covariate influence in multivariate datasets by defining a covariance weight vector **w<sub>CP</sub>** that is multiplied with the centered augmented data matrix **X<sub>aug</sub>** that contains **X**, **y** and also the covariates **z** (**X<sub>aug</sub>** = [**X**, **y** ,**z**]). The weight vector **w<sub>CP</sub>** has as many entries as variables exist in **X<sub>aug</sub>** and contains zeros in positions of **X** and **y** and ones in **z** position (= covariate entry). This enables the calculation of the covariance projection scores **t<sub>CP</sub>** and subsequently the loadings **p<sub>CP</sub><sup>T</sup>**. 

1) **t<sub>CP</sub>** = **X<sub>aug</sub>  w<sub>CP</sub>** <br>
2) **p<sub>CP</sub><sup>T</sup>** = **t<sub>CP</sub><sup>T</sup>** **X<sub>aug</sub>** / (**t<sub>CP</sub><sup>T</sup>** **t<sub>CP</sub>**)

Using the loadings and scores, the adjusted augmented **E<sub>aug</sub>** can be calculated (= **Dataset after covariate projection** in *mvpaShiny* covariate projection tab). 

3) **E<sub>aug</sub>** = **X<sub>aug</sub>**  –  **t<sub>CP</sub>** **p<sub>CP</sub><sup>T</sup>**

This matrix can be used to perform follow-up analyses like PLS-regression. Ideally, the influence of the covariates have been removed so that the "real" relationship between explanatory and response variables can be determined.

![covariate_projection_main](assets/images/Covariate projection/covariate_projection_main.png)

In order to perform the covariance projection in *mvpaShiny* a dataset needs to be selected (= **X<sub>aug</sub>**) and the covariates have to be defined. Standardization is a useful and recommended step to equalize variables importance. The **Perform covariate projection** button will start the calculation.

> NOTE: The **order** of selected covariates matters when the explained variance by covariate plot is viewed. While **E<sub>aug</sub>** will be identical, the explained variance by the individual covariate might differ.



### Explained variance by covariate plot

![confounder_projection_explained_variance](assets/images/Covariate projection/covariate_projection_explained_variance-165714781316713.png)
