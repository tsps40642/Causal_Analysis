INTRODUCTION_TO_CAUSAL_ANALYSIS_AND_EXPERIMENT
================

# How to establish causality

1.  Experiment: A/B testing, randomized control trials (RCT), etc.  
2.  Observational techniques: matching techniques, panel data, etc.

### Experiment

Steps for experiments:  
1. Take a population  
2. Randomize each person between treatment and control groups  
- If subjects randomized properly, treatment and control groups should
have similar demographics  
3. Analyze data

# Codes

In the “who likes you feature” example, does the feature increase
matches?

### Statistical summary

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
focal_user_df_females <- read.csv('G:/My Drive/UMN Courses/6441/focal_user_df_females.csv')
```

Check the user number of treatment and control group

``` r
focal_user_df_females %>% group_by(manipulation) %>% summarise(n()) # n() gives the count number 
```

    ## # A tibble: 2 × 2
    ##   manipulation `n()`
    ##          <int> <int>
    ## 1            0  5557
    ## 2            1  5614

``` r
# roughly the same number of users in both treatment and control groups
```

Create new column: total matches = message sent + message received

``` r
# January 
focal_user_df_females = focal_user_df_females %>% mutate(total_1 = match_sent_cnt_1 + match_rcvd_cnt_1)

# February 
focal_user_df_females = focal_user_df_females %>% mutate(total_2 = match_sent_cnt_2 + match_rcvd_cnt_2)
```

Check the average user ages of treatment and control group

``` r
focal_user_df_females %>% group_by(manipulation) %>% summarise(mean(age_raw))
```

    ## # A tibble: 2 × 2
    ##   manipulation `mean(age_raw)`
    ##          <int>           <dbl>
    ## 1            0            33.2
    ## 2            1            33.0

Do the hypothesis testing to see if the average age is the same between
control and treatment groups

``` r
# H0: same 
# H1: not the same 

t.test(age_raw ~ manipulation, data = focal_user_df_females)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  age_raw by manipulation
    ## t = 0.83986, df = 11159, p-value = 0.401
    ## alternative hypothesis: true difference in means between group 0 and group 1 is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.2465185  0.6161296
    ## sample estimates:
    ## mean in group 0 mean in group 1 
    ##        33.21130        33.02649

``` r
# p-value = 0.401, no sufficient evidence to reject the same  
```

Do the hypothesis testing to see whether being a white has relationship
to total matches

``` r
# H0: same 
# H1: not the same 

t.test(total_2 ~ as.factor(white), data = focal_user_df_females) 
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  total_2 by as.factor(white)
    ## t = -0.15352, df = 3660.7, p-value = 0.878
    ## alternative hypothesis: true difference in means between group 0 and group 1 is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.5706084  0.4877377
    ## sample estimates:
    ## mean in group 0 mean in group 1 
    ##        4.019053        4.060488

``` r
# p-value = 0.878, no sufficient evidence to reject the same  
```

Do the hypothesis testing to see whether total matches differ between
treatment and control groups

``` r
# H0: matches the same 
# H1: matches not the same 

t.test(total_2 ~ as.factor(manipulation), data = focal_user_df_females)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  total_2 by as.factor(manipulation)
    ## t = -3.0536, df = 11031, p-value = 0.002267
    ## alternative hypothesis: true difference in means between group 0 and group 1 is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.7237143 -0.1578297
    ## sample estimates:
    ## mean in group 0 mean in group 1 
    ##        2.675544        3.116316

``` r
# p-value = 0.002267, sufficient evidence to reject the same >> accept 
```

You can also build a lm and check by its summary

``` r
# H0: β_1 = 0
# H1: β_1 != 0 

summary(lm(total_2 ~ as.factor(manipulation), data = focal_user_df_females))
```

    ## 
    ## Call:
    ## lm(formula = total_2 ~ as.factor(manipulation), data = focal_user_df_females)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ##  -3.116  -3.116  -2.676  -0.676 143.884 
    ## 
    ## Coefficients:
    ##                          Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                2.6755     0.1024  26.131  < 2e-16 ***
    ## as.factor(manipulation)1   0.4408     0.1444   3.052  0.00228 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 7.633 on 11169 degrees of freedom
    ## Multiple R-squared:  0.0008331,  Adjusted R-squared:  0.0007437 
    ## F-statistic: 9.313 on 1 and 11169 DF,  p-value: 0.002281

``` r
# whether using as.factor(manipulation) or manipulation is the same result
# p-value = 0.002281, sufficient evidence to reject the same >> accept 
```

# Some important things in experiment

### Partial differential

Coefficient of manipulation (0.4408) is the partial differential of
matches over treatment, that is, the average change in matches between
the control and treatment group

### What is contained in the error term?

1.  In experiment, the error term captures all other things we’re not
    interested in  
2.  Note that this is different from what we do in statistical inference
    where the error term indicates the uncertainty of the model!

### Assumption on the error term

1.  In experiment, corr(treatment, error) ~= 0  
2.  Compared to the error in linear model:
    1.  Y is independent of errors  
    2.  error ~ i.i.d. N(0, σ), assumed normality of errors  
    3.  variance of the residuals is the same for all values of
        predictors, i.e. no heteroskedasticity

### One treatment at a time

Note that we will include many predictors in the lm, yet we only have a
specific interest in ONE treatment at a time in causal inference

### Causality vs. prediction

1.  In prediction, we want the predictors with the highest explanatory
    power \>\> max R_squared  
2.  In causal inference, we are interested in estimating effect of
    intervention  

- No need to include explanatory predictors to boost R_squared  
- No need to understand distribution of the residuals since we’re not
  assessing model fit  
- No need to do train-testing split

# Heterogeneity analysis

Think about how two factors can interact.

1.  The interaction term in the lm is the additional lift due to
    combining two factors  
2.  That is, the interaction term gives you the change in the dependent
    variable one can expect if both treatments are present
    simultaneously  
3.  Heterogeneity analysis allows us to identify where the effect is
    located
