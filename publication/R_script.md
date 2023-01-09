---
layout: default
title: mvpa R workflow
parent: Publication
nav_order: 1
---

# The mvpa R workflow: HOMO-IR case study

The herein shown code snippets shall highlight how the mvpa R package can be used and follows the narrative of the corresponding publication.

## Setup 

We first need to load the packages and make the dataset available

```R
library(mvpa)
library(dplyr)
library(plotly)

dataset <- mvpa::HOMA_IR

```

We then standardize all explanatory variables, but keep the response (HOMA-IR) unstandardized

```R
selected_for_standardization <- colnames(dataset)[-1]
dataset_standardized <- mvpa::scale_data(data = dataset,
                                         col_names = selected_for_standardization,
                                         method = 'sd')

# # Variances for each variable
round(apply(dataset_standardized, 2,var), 3)

```

## Figure 2

We then create the dataset for the first step, where we perform dimension reduction, using PLS-R (partial least squares regression). We want to represent the 26 lipoproteins by the single single target scores vector **t<sub>TP</sub>**. More theory on PLS-R and target projection can be found [here](mvpaShiny/PLS-Regression.md).

```R
# # Response: HOMA_IR
# # Explanatory variables: Lipoproteins (26 variables)
# #
# # Optimal number of components: 4
# # Explained variance in the response: 24.4%
# # Lipoprotein target component (tTP) explains 32.7%

lipoproteins <- colnames(dataset)[4:29]

figure_2_dataset <- dataset_standardized[c('HOMA_IR', lipoproteins)]

# Perform repeated PLS-R, using Monte-Carlo resampling to identify optimal number of components
set.seed(55)
figure_2_result <- mvpa::perform_mc_pls_tp(data = figure_2_dataset,
                                           response = 'HOMA_IR',
                                           nr_components = 6,
                                           mc_resampling = TRUE,
                                           nr_repetitions = 1000,
                                           cal_ratio = 0.5,
                                           validation_threshold = 0.5,
                                           standardize = FALSE)

```

We created a list element that contains information about the created PLS model, but also contains the target projection values, such as the Selectivity ratio (SR) or Selectivity fraction (SF).

One information that we can find is the optimal number of components that was determined, using the Monte Carlo resampling methodology. We can also visualize the model error (cost function) for different numbers of components which helps us understand, why the chosen number of components can be considered ideal. We simply pass the PLS-R result to the *plot_cost_function_values_distribution()* function. 

```R
# Optimal number of components
figure_2_result$A_optimal

# Validation plot(s)
set.seed(55)
figure_2 <- mvpa::plot_cost_function_values_distribution(figure_2_result, show_title = FALSE)
```

<iframe src="/mvpaShiny_documentation/publication/html/figure_2.html" height=400px width="100%" style="border:none;"></iframe>

Figure 2. Validation plot that indicates that 4 PLS components are the optimal choice for this dataset.



Further information that we can retrieve of the model is the explained variance in the response and the explanatory variables, and the target score **t<sub>TP</sub>**. The target score is derived from the PLS model, using the optimal number of components.

```R
# Explained variance in y in %
# By component
figure_2_result$explained_variance_in_y * 100

# Total explained variance in y
sum(figure_2_result$explained_variance_in_y) * 100

# Explained variance by the lipoprotein target component tTP
figure_2_result$explained_variance_tTP * 100
```



## The enhanced standardized HOMO-IR dataset

We now need to enhance the original standardized dataset by adding the target score to the dataset.

```R
# Add target score tTP to full dataset
enhanced_standardized_dataset <- mvpa::enhance_dataset(data = dataset_standardized,
                                                       data_to_add = figure_2_result$target_projection_A_opt$tTP)
```

This dataset forms the foundation of all upcoming analyses.



## Table 1

At first the dataset is inspection by means of a pairwise correlation analysis.  A subset of variables is selected. This produces a correlation matrix that can also be visualized. Only one triangle of the symmetric matrix is shown.

```R
table_1_variables <- c('Age', 'Sex', 'tTP', 'HOMA_IR')

# Calculate pairwise correlation matrix for selected variables
table_1_result <- mvpa::correlation(data=enhanced_standardized_dataset,
                                    correlate = 'variables',
                                    selection = table_1_variables)

# The correlation matrix
table_1 <- table_1_result$correlation_matrix

# The correlation matrix as a plot
table_1_plot <- mvpa::plot_correlation(table_1_result)

```

<iframe src="/mvpaShiny_documentation/publication/html/table_1_plot.html" height=400px width="100%" style="border:none;"></iframe>



## Covariate projections for figure 3, 4 and table 2

In the next step, we perform the covariate projection and project certain variables out of the enhanced standardized dataset. We start with the variable 'Age'.

```R
# Adjust for Age
PA_values <- colnames(dataset)[33:55]
adiposity_values <- colnames(dataset)[30:32]

age_adjustment <- mvpa::perform_covariate_projection(X_aug = enhanced_standardized_dataset,
                                                     covariates = 'Age',
                                                     standardize = TRUE)

```

We now redo this step two times with more variables. This step is order-sensitive - choosing c('Age', 'Sex') will not produce the same results as c('Sex', 'Age').

```R
# Adjust for Age and Sex
age_sex_adjustment <- mvpa::perform_covariate_projection(X_aug = enhanced_standardized_dataset,
                                                         covariates = c('Age', 'Sex'),
                                                         standardize = TRUE)

# Adjust for Age, Sex and lipoproteins (as tTP - target component)
age_sex_lipoproteins_adjustment <- mvpa::perform_covariate_projection(X_aug = enhanced_standardized_dataset,
                                                                      covariates = c('Age', 'Sex', 'tTP'),
                                                                      standardize = TRUE,
                                                                      reorder_by_covariates = TRUE)
```

We now combine all three individual adjustments into one table. This will be needed for table 2 later on.

```R
combined_adjustment_table <- rbind(age_adjustment$explained_variance_by_covariates[2, c('HOMA_IR', adiposity_values, PA_values)],
                                   age_sex_adjustment$explained_variance_by_covariates[3, c('HOMA_IR', adiposity_values, PA_values)],
                                   age_sex_lipoproteins_adjustment$explained_variance_by_covariates[4, c('HOMA_IR', adiposity_values, PA_values)])
```



## Figure 3

We now create the plot that indicates the explained variance by each selected covariate.

```R
figure_3 <- mvpa::plot_explained_var_by_covariates(age_sex_lipoproteins_adjustment, rotate = TRUE)

# Tabulated version of Figure 3.
figure_3_table <- age_sex_lipoproteins_adjustment$explained_variance_by_covariates
```

<iframe src="/mvpaShiny_documentation/publication/html/figure_3.html" height=400px width="100%" style="border:none;"></iframe>



## Figure 4 a)

We now use the adjusted datasets and create validated PLS-R models and perform target projection. By means of the target projection, we can retrieve selectivity fractions (SF) that help us identify variables that explain the chosen response the most (here HOMA-IR).

```R
# a) Unadjusted data
figure_4_a_dataset <- dataset_standardized[c('HOMA_IR', adiposity_values, PA_values)]

figure_4_a_result <- mvpa::perform_mc_pls_tp(data = figure_4_a_dataset,
                                                    response = 'HOMA_IR',
                                                    mc_resampling = TRUE,
                                                    nr_components = 8,
                                                    nr_repetitions = 1000,
                                                    cal_ratio = 0.5,
                                                    validation_threshold = 0.5,
                                                    standardize = FALSE)

# Selectivity fraction (SF) plot
figure_4_a <- mvpa::plot_tp_value_mc(result_list = figure_4_a_result,
                                     tp_value_to_plot = 'selectivity_fraction',
                                     component = figure_4_a_result$A_optimal,
                                     confidence_limits = c(0.025, 0.975))

```

<iframe src="/mvpaShiny_documentation/publication/html/figure_4_a.html" height=400px width="100%" style="border:none;"></iframe>



## Figure 4 b)

```R
# b) Adjusted data (adjusted for age and sex)
figure_4_b_dataset <- age_sex_adjustment$residual_variance_df[c('HOMA_IR', adiposity_values, PA_values)]

# PLS-R
# Response: HOMA_IR
# Explanatory variables: Adiposity values and PA values after adjustment for age, sex
# Optimal number of components: 7
set.seed(55)
figure_4_b_result <- mvpa::perform_mc_pls_tp(data = figure_4_b_dataset,
                                             response = 'HOMA_IR',
                                             mc_resampling = TRUE,
                                             nr_components = 8,
                                             nr_repetitions = 1000,
                                             cal_ratio = 0.5,
                                             validation_threshold = 0.5,
                                             standardize = FALSE)

# Selectivity fraction (SF) plot
figure_4_b <- mvpa::plot_tp_value_mc(result_list = figure_4_b_result,
                                     tp_value_to_plot = 'selectivity_fraction',
                                     component = figure_4_b_result$A_optimal,
                                     confidence_limits = c(0.025, 0.975))

```

<iframe src="/mvpaShiny_documentation/publication/html/figure_4_b.html" height=400px width="100%" style="border:none;"></iframe>



## Figure 4 c)

```R
# c) Adjusted data (adjusted for age and sex and lipoproteins target score (tTP)
figure_4_c_dataset <- age_sex_lipoproteins_adjustment$residual_variance_df[c('HOMA_IR', adiposity_values, PA_values)]

# PLS-R
# Response: HOMA_IR
# Explanatory variables: Adiposity values and PA values after adjustment for age, sex and lipoproteins target score (tTP)
# Optimal number of components: 7
set.seed(55)
figure_4_c_result <- mvpa::perform_mc_pls_tp(data = figure_4_c_dataset,
                                             response = 'HOMA_IR',
                                             mc_resampling = TRUE,
                                             nr_components = 8,
                                             nr_repetitions = 1000,
                                             cal_ratio = 0.5,
                                             validation_threshold = 0.5,
                                             standardize = FALSE)

# Selectivity fraction (SF) plot
figure_4_c <- mvpa::plot_tp_value_mc(result_list = figure_4_c_result,
                                     tp_value_to_plot = 'selectivity_fraction',
                                     component = figure_4_c_result$A_optimal,
                                     confidence_limits = c(0.025, 0.975))			HTML
```

<iframe src="/mvpaShiny_documentation/publication/html/figure_4_c.html" height=400px width="100%" style="border:none;"></iframe>



## Table 2

We will now produce an overview table of the explained variances before and after the adjustments. The table generation is split into two parts a and b.

```R
# a - Part - Percent remaining variance of total variance after projections (adjustments)
table_2_a <- sapply(list('HOMA_IR', adiposity_values, PA_values),
                    FUN = function(x) apply(combined_adjustment_table[x] * 100, 1, mean))

# add unadjusted data - simply add 100%, since no adjustment has been performed at this point
table_2_a <- rbind(rep(100, 3), table_2_a)

colnames(table_2_a) <- c('V(HOMA_IR)a', 'V(Adiposity)a', 'V(PA)a')
```

B part of the overview table.

```R
# b - Part - Percent explained variance in lipoproteins and HOMA-IR of their original total variance (before any adjustments)
table_2_b_dataset_age_adjusted <- age_adjustment$residual_variance_df[c('HOMA_IR', adiposity_values, PA_values)]

# PLS-R
# Response: HOMA_IR
# Explanatory variables: Adiposity values and PA values after adjustment for age
# Optimal number of components: 7
set.seed(55)
pls_model_age_adjusted_result <- mvpa::perform_mc_pls_tp(data = table_2_b_dataset_age_adjusted,
                                                         response = 'HOMA_IR',
                                                         mc_resampling = TRUE,
                                                         nr_components = 8,
                                                         nr_repetitions = 1000,
                                                         cal_ratio = 0.5,
                                                         validation_threshold = 0.5,
                                                         standardize = FALSE)
# Optimal number of components
pls_model_age_adjusted_result$A_optimal

# All models (age-adjusted, age-sex-adjusted, age-sex-lipoproteins-adjusted) for Table 2
models_table_2 <- list(figure_4_a_result, pls_model_age_adjusted_result, figure_4_b_result, figure_4_c_result)

# Normalize the explained variance in the response, using percent of the remaining variance after the adjustments (-> V(HOMA_IR)a)
# This step is crucial and needs to be considered, when working with adjusted datasets!
explained_variances_in_response <- sapply(models_table_2, FUN = function(x) sum(x[['explained_variance_in_y']]))
V_HOMA_IR_a <- table_2_a[, 1]

V_HOMA_IR_b <- explained_variances_in_response * V_HOMA_IR_a

# Grab the averaged explained variances for the variables belonging to the groups adiposity (3 variables)
# and physical activity (23 variables)
V_adiposity_b <- sapply(models_table_2, FUN = function(x) mean(x[['target_projection_A_opt']][['explained_variance']][adiposity_values, ]) * 100)
V_PA_b <- sapply(models_table_2, FUN = function(x) mean(x[['target_projection_A_opt']][['explained_variance']][PA_values, ]) * 100)

table_2_b  <- data.frame('V(HOMA_IR)b' = V_HOMA_IR_b,
                         'V(Adiposity)b' = V_adiposity_b,
                         'V(PA)b' = V_PA_b,
                         check.names = FALSE)
```

We combine the a and b part of the table.

```R
# Table 2 (table 2a and 2b combined)
table_2 <- round(cbind(table_2_a, table_2_b), 1)
rownames(table_2) <- c('Unadjusted', 'Age', 'Age and sex', 'Age, sex and lipoproteins')
```

|                           | V(HOMA_IR)a | V(Adiposity)a | V(PA)a | V(HOMA_IR)b | V(Adiposity)b | V(PA)b |
| ------------------------- | ----------- | ------------- | ------ | ----------- | ------------- | ------ |
| Unadjusted                | 100.0       | 100.0         | 100.0  | 30.3        | 72.2          | 16.4   |
| Age                       | 99.9        | 99.9          | 99.8   | 30.2        | 72.4          | 16.4   |
| Age and sex               | 96.8        | 95.9          | 95.8   | 27.1        | 73.6          | 13.2   |
| Age, sex and lipoproteins | 74.3        | 81.1          | 92.0   | 13.1        | 54.1          | 7.5    |



## Figure 5

In the last experiment, we go back to the enhanced full dataset (ie. includes the target score **t<sub>TP</sub>** and the 26 lipoproteins) and perform covariate projection.
Age, sex and the target projection score **t<sub>TP</sub>** will be used as covariates.

```R
full_dataset_adjustment <- mvpa::perform_covariate_projection(X_aug = enhanced_standardized_dataset,
                                                              covariates = c('Age', 'Sex', 'tTP'),
                                                              standardize = TRUE)

full_dataset_adjusted <- mvpa::remove_variables(data = full_dataset_adjustment$residual_variance_df,
                                                vars_to_remove = c('Age', 'Sex', 'tTP'))

# Reorder variables for plotting
figure_5_dataset <- full_dataset_adjusted[c('HOMA_IR', adiposity_values, lipoproteins, PA_values)]

```

After the adjustment, PLS-R is performed.

```R
# PLS-R
# Response: HOMA_IR
# Explanatory variables: Adiposity values, PA values, lipoproteins
#                        after adjustment for age, sex and the lipoprotein target component
# Optimal number of components: 3
set.seed(55)
figure_5_result <- mvpa::perform_mc_pls_tp(data = figure_5_dataset,
                                           response = 'HOMA_IR',
                                           mc_resampling = TRUE,
                                           nr_components = 8,
                                           nr_repetitions = 1000,
                                           cal_ratio = 0.5,
                                           validation_threshold = 0.5,
                                           standardize = FALSE)

# Optimal number of components
figure_5_result$A_optimal

# Normalize the explained variance in the response, using the percent of the remaining variance after the adjustment
# (-> V(HOMA_IR)a = 74.3%, adjusted for age, sex and the lipoproteins which are represented by the target score tTP)
figure_5_explained_variance_normalized <- sum(figure_5_result$explained_variance_in_y) * table_2['Age, sex and lipoproteins', 'V(HOMA_IR)a']

# Selectivity fraction (SF) plot
figure_5 <- mvpa::plot_tp_value_mc(result_list = figure_5_result,
                                   tp_value_to_plot = 'selectivity_fraction',
                                   component = figure_5_result$A_optimal,
                                   confidence_limits = c(0.025, 0.975),
                                   rotate = TRUE)

explained_variance_lipoproteins <- mean(figure_5_result$target_projection_A_opt$explained_variance[lipoproteins, ]) * 100
explained_variance_PA <- mean(figure_5_result$target_projection_A_opt$explained_variance[PA_values, ]) * 100
explained_variance_adiposity <- mean(figure_5_result$target_projection_A_opt$explained_variance[adiposity_values, ]) * 100
```

<iframe src="/mvpaShiny_documentation/publication/html/figure_5.html" height=400px width="100%" style="border:none;"></iframe>

Figure 5. The association patterns with data adjusted for age, sex and the lipoprotein target component associated to HOMA-IR and with the 26 lipoprotein features included as explanatory variables.
