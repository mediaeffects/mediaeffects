---
title: "mediaeffects: Modern R toolkit for understanding media effects"
tags:
  - R
  - media effects
  - heterogeneity
  - causal forests
  - machine learning
authors:
  - name: Saurabh Khanna
    orcid: 0000-0002-9346-4896
    affiliation: 1
  - name: Jochen Peter
    orcid: 0000-0002-2356-6619
    affiliation: 1
affiliations:
 - name: Amsterdam School of Communication Research, University of Amsterdam
   index: 1
citation_author: Khanna and Peter
date: 12 January 2026
year: 2026
bibliography: references.bib
output: rticles::joss_article
csl: apa.csl
journal: JOSS
header-includes: 
  - \newcounter{none}
  - \providecommand{\pandocbounded}[1]{#1}
  - \usepackage{bookmark}
  - \hypersetup{bookmarks=true, bookmarksnumbered=true, bookmarksopen=true, bookmarksdepth=3}
---








# Introduction

Media effects research has traditionally focused on estimating average treatment effects (ATEs) of media exposure on various outcomes. However, there is growing recognition that, more often than not, these effects may not be uniform across all individuals. Understanding heterogeneity in media effects is crucial for tailoring interventions and policies to specific sub-populations. Modern machine learning (ML) techniques, particularly causal forests, offer powerful tools to explore and estimate heterogeneous treatment effects (HTEs) in observational data. Causal forests extend traditional random forests by focusing on estimating conditional average treatment effects (CATEs) while accounting for confounding variables.

We define heterogeneity in media effects as the variation in the effect of media exposure on an outcome across different individuals or subgroups, influenced by their unique characteristics or contexts. This paper introduces the `mediaeffects` R package, which implements the causal forest algorithm, tailored to data distributions prevalent in communication research, to explore heterogeneity in media effects. We provide an illustrative example using data from the 2024 General Social Survey to demonstrate how researchers can leverage this package to uncover nuanced insights into media effects.

# Current approach to exploring heterogeneity

Researchers have traditionally used two approaches to explore heterogeneity in media effects.

## Subgroup analyses

The first approach consists of splitting the sample into subgroups based on certain participant characteristics (such as, age, gender, education level) or media characteristics (such as, type of media content, frequency of exposure), and estimating the treatment effect separately within each subgroup. This approach allows researchers to identify whether the media effect varies across different segments of the population. Let us take the example of exploring heterogeneity in the effect of watching conservative news channels on support for Donald Trump. Researchers may split the sample into age groups (e.g., younger vs. older adults) and estimate the effect of watching conservative news channels on Trump support within each age group separately. If the effect is significantly stronger among older adults compared to younger adults, this would suggest heterogeneity in the media effect based on age.

Mathematically, this can be represented as:

\[
\begin{aligned}
Y_i = \beta_0 + \beta_1 \cdot Treatment_i + \epsilon_i
\end{aligned}
\]

where $Y_i$ is the rating of Trump for individual $i$, and $Treatment_i$ indicates exposure to conservative news.

for each subgroup \(g\):

\[
\begin{aligned}
Y_{i,g} = \beta_{0,g} + \beta_{1,g} \cdot Treatment_{i,g} + \epsilon_{i,g}
\end{aligned}
\]

where \(\beta_{1,g}\) represents the effect of watching conservative news channels on Trump support within subgroup \(g\). By comparing \(\beta_{1,g}\) across different subgroups, researchers can assess whether there is heterogeneity in the media effect.

## Linear regression based interactions

The second approach involves incorporating interaction terms into linear regression models. By including interactions between the treatment variable (media exposure) and potential moderators (participant or media characteristics), researchers can assess whether the effect of media exposure on the outcome differs based on these moderators. Picking up on our example, researchers may include an interaction term between watching conservative news channels and age in a regression model predicting Trump support. A significant interaction term would indicate that the effect of watching conservative news channels on Trump support varies by age.

Mathematically, this can be represented as:

\[
\begin{aligned}
Y_i = \beta_0 &+ \beta_1 W_i + \beta_2 X_i \\
&+ \beta_3 (W_i \times X_i) + \epsilon_i
\end{aligned}
\]

where $W_i$ is the treatment (conservative news exposure), $X_i$ is the moderator (age), and their interaction $W_i \times X_i$ tests for heterogeneity.

where \(\beta_3\) captures the heterogeneity in the media effect based on age. The significance and magnitude of \(\beta_3\) indicate whether and how the effect of watching conservative news channels on Trump support varies with age.

## Limitations

<!-- Paragraph 1 (Pre-specification): Researchers must specify moderators before analysis, which pro- -->
<!-- tects against data-driven false positives but risks missing important moderators that theory did not -->
<!-- anticipate. -->
<!-- Paragraph 2 (Multiple comparisons): Testing many moderators at α = .05 inflates false positive -->
<!-- rates (e.g., 40% chance of one spurious significant result when testing 10 moderators). Researchers -->
<!-- face a tradeoff between examining few moderators (missing discoveries) or many (false positives or -->
<!-- reduced power after corrections). -->
<!-- Paragraph 3 (Linearity assumption): The interaction term W × X assumes treatment effects change -->
<!-- linearly with moderator values. Nonlinear patterns (e.g., effects only at extreme moderator levels) -->
<!-- cannot be captured. -->
<!-- Paragraph 4 (Average effects only): Traditional approaches estimate group-level averages rather than -->
<!-- individual-level effects, limiting precision in understanding who is most affected. -->
<!-- Paragraph 5 (No heterogeneity test): There is no formal test for whether heterogeneity exists inde- -->
<!-- pendent of whether specific hypothesized moderators are significant. -->

While these traditional approaches have been widely used, they come with several limitations. First, they require researchers to pre-specify potential moderators based on theory or prior research, which may lead to overlooking important moderators that were not anticipated. Second, testing multiple moderators increases the risk of false positives due to multiple comparisons, making it challenging to balance between exploring many moderators and maintaining statistical power. Third, linear regression-based interactions assume a linear relationship between the moderator and treatment effect, which may not capture more complex, nonlinear patterns of heterogeneity. Fourth, these approaches typically estimate average effects within subgroups rather than individual-level treatment effects, limiting the precision of understanding who is most affected by media exposure. Finally, there is no formal statistical test for the presence of heterogeneity itself, independent of specific hypothesized moderators.

# Modern ML based approaches

Recent advances in machine learning (ML) have introduced new methods for exploring heterogeneity in treatment effects. Causal forests, an extension of random forests, have emerged as a powerful tool for estimating conditional average treatment effects (CATEs) while accounting for confounding variables. Unlike traditional approaches, causal forests do not require pre-specification of moderated relationships, and can automatically identify complex interactions and nonlinear relationships in the data. This allows researchers to uncover nuanced patterns of heterogeneity that may be missed by traditional methods.

## Conceptual foundations

In order to understand causal forests, it is important to first understand decision trees and random forests. We will briefly review these concepts below.

## Decision trees

<!-- accessible explanation for a total beginner with no idea of ML concepts -->

A decision tree is a simple yet powerful machine learning algorithm used for both classification and regression tasks. It works by recursively splitting the data into subsets based on the values of input features, creating a tree-like structure where each internal node represents a decision based on a feature, each branch represents the outcome of that decision, and each leaf node represents a final prediction or outcome.

Using our example of exploring heterogeneity in the effect of watching conservative news channels on support for Donald Trump, a decision tree would start with the entire dataset and look for the feature (e.g., age, education level, political ideology) that best splits the data into two groups that are most different in terms of Trump support. This process is repeated recursively for each subgroup until a stopping criterion is met (e.g., minimum number of observations in a leaf node).

Mathematically, a decision tree can be represented as a series of binary splits based on feature values. For example, the first split might be based on age:

\[ \text{If } age < 40 \text{ then go to left subtree else go to right subtree} \]

Each subsequent split would further divide the data based on other features, ultimately leading to leaf nodes that provide predictions for Trump support based on the characteristics of individuals in that leaf.

## Random forests

A random forest is an ensemble learning method that combines multiple decision trees to improve predictive accuracy and reduce overfitting. In a random forest, a large number of decision trees are trained on different subsets of the data (using bootstrapping) and different subsets of features (using random feature selection). The final prediction is made by aggregating the predictions from all individual trees, typically through majority voting for classification tasks or averaging for regression tasks. 

Using our example, a random forest would create multiple decision trees, each trained on a different random sample of the data and using different random subsets of features. Each tree would make its own prediction for Trump support based on the characteristics of individuals. The final prediction for each individual would be the average of the predictions from all trees in the forest. 

Mathematically, the prediction from a random forest can be represented as:
\[ \hat{y}_i = \frac{1}{T} \sum_{t=1}^{T} \hat{y}_{i}^{(t)} \]

where \( \hat{y}_i \) is the final prediction for individual \(i\), \(T\) is the total number of trees in the forest, and \( \hat{y}_{i}^{(t)} \) is the prediction from tree \(t\) for individual \(i\).

## Causal forests

Causal forests extend the random forest framework to estimate heterogeneous treatment effects (HTEs) by focusing on the conditional average treatment effect (CATE) for each individual, rather than just predicting outcomes. Causal forests are designed to handle observational data where treatment assignment may be confounded by other variables. They achieve this by using a two-stage approach: first, they estimate the propensity score (the probability of receiving treatment given covariates), and then they use this information to adjust the splits in the decision trees to focus on estimating treatment effects.

Using our example, a causal forest would create multiple decision trees that are specifically designed to estimate the effect of watching conservative news channels on Trump support, while accounting for confounding variables such as age, education level, and political ideology. Each tree would estimate the treatment effect for individuals based on their characteristics, and the final CATE for each individual would be obtained by averaging the treatment effect estimates from all trees in the forest.

Mathematically, the CATE for individual \(i\) can be represented as:

\[ \hat{\tau}(X_i) = \frac{1}{T} \sum_{t=1}^{T} \hat{\tau}^{(t)}(X_i) \]

where \( \hat{\tau}(X_i) \) is the estimated CATE for individual \(i\), \(T\) is the total number of trees in the causal forest, and \( \hat{\tau}^{(t)}(X_i) \) is the treatment effect estimate from tree \(t\) for individual \(i\).

# Illustrative example using `mediaeffects` R package

We use the 2024 General Social Survey (GSS) data with a searchable codebook [here](https://gssdataexplorer.norc.org/variables/vfilter). We focus on exploring heterogeneity in the effect of conservatism on support for Donald Trump, controlling for a wide range of demographic, socioeconomic, and attitudinal covariates. 




## Exploration phase

### Pre-analysis checks

Before running the causal forest, it is important to check for high correlations among covariates, as highly correlated variables can lead to instability in the estimates. We identify pairs of variables with high correlations (e.g., |r| >= 0.7).


``` r
# Check for highly correlated covariates (|r| >= 0.7)
highly_correlated(df_clean, threshold = 0.7)
```

```
## # A tibble: 37 x 3
##    x        y                    r
##    <chr>    <chr>            <dbl>
##  1 age      cohort          -0.996
##  2 coninc   realinc          0.989
##  3 id       vstrat           0.981
##  4 prestg10 prestg105plus    0.980
##  5 sei10    sei10inc         0.943
##  6 sei10    sei10educ        0.932
##  7 degree   attend_college   0.900
##  8 padeg    father_college   0.894
##  9 madeg    mother_college   0.882
## 10 maborn   parents_born_us  0.868
## # i 27 more rows
```


We define our outcome variable (Y) as the rating of Donald Trump, our treatment variable (W) as conservatism, and our covariate matrix (X) by selecting relevant demographic, socioeconomic, and attitudinal variables from the cleaned dataset.


``` r
# Define outcome (Y), treatment (W), and covariates (X) for causal forest
# Y: Rating of Donald Trump (0-100 scale)
# W: Conservative ideology (binary: 1 = conservative, 0 = not conservative)
# X: Demographic, socioeconomic, and attitudinal covariates

Y <- df_clean$rating_trump
W <- df_clean$conservative
X <- 
  df_clean %>%
  select(all_of(c(
    # Demographics
    "female", "white", "black", "hispanic", "age", "lgbt",
    # Socioeconomic
    "educ", "income16", "madeg", "padeg", "prestg10",
    # Family & household
    "married", "adults", "childs",
    # Geographic & background
    "rural", "midwest", "mobile16", "born",
    # Religion & engagement
    "attend", "pray",
    # Attitudes & behaviors
    "lkelyvot", "compuse", "othlang", "diagnosd",
    "raclive", "vetyears",
    # Wellbeing
    "happy", "health", "finalter", "satfin", "satdemoc"
  )))
```

Now that we have defined our variables, we can proceed to estimate heterogeneity in the effect of conservatism on Trump ratings using causal forests [@wager2018estimation].


### Step 1: Does heterogeneity exist?

We can start by testing for the presence of heterogeneity in the treatment effect using the RATE (Ranked Average Treatment Effect) method. The method works by ranking individuals based on their estimated treatment effects and then calculating the average treatment effect for different quantiles of this ranking. If the treatment effect varies significantly across these quantiles, it indicates the presence of heterogeneity. In other words, if the targeting curve (RATE curve) is above the diagonal line, it suggests that there is heterogeneity in the treatment effect. And in terms of the AUTOC (Area Under the Targeting Operator Characteristic Curve) metric, a value significantly greater than 0 indicates the presence of heterogeneity.

We use the `test_heterogeneity` function from the `mediaeffects` package to perform this test. The function takes in the outcome variable (Y), treatment variable (W), covariate matrix (X), and survey weights (if applicable) as inputs, and returns the RATE curve and AUTOC estimate.


``` r
# Test for heterogeneity using RATE method
het_test <- test_heterogeneity(
  Y, W, X, 
  weights = df_clean$wtssps
)

# Plot the targeting curve
plot(het_test$rate, 
     main = "Targeting Operator Characteristic")
```

![](paper_files/figure-latex/test-heterogeneity-1.pdf)<!-- --> 

We can see from the plot that the targeting curve is above the diagonal, indicating that there is heterogeneity in the effect of conservatism on Trump ratings.


``` r
# Print AUTOC estimate and 95% CI
autoc_val <- round(het_test$autoc, 2)
autoc_ci <- round(1.96 * het_test$std_err, 2)
print(paste("AUTOC:", autoc_val, "+/-", autoc_ci))
```

```
## [1] "AUTOC: 12.65 +/- 3.89"
```

The AUTOC is significantly greater than 0, indicating that there is significant heterogeneity in the effect of conservatism on Trump ratings. This suggests that the effect of conservatism on Trump ratings varies across individuals, and we can proceed to estimate the conditional average treatment effects (CATEs) for each individual.

### Step 2: Individual level effects sizes

We can estimate the CATEs for each individual using the `estimate_cates` function from the `mediaeffects` package. This function fits a causal forest model to the data and estimates the treatment effect for each individual based on their covariate values. In the context of this example, it provides an estimate of how much being conservative affects an individual's rating of Donald Trump, taking into account their unique characteristics. 

We use the `estimate_cates` function to obtain the CATEs for all observations in our dataset. The function takes in the outcome variable (Y), treatment variable (W), covariate matrix (X), and survey weights (if applicable) as inputs, and returns the estimated CATEs along with the average treatment effect (ATE) and its standard error.


``` r
# Estimate individual-level treatment effects (CATEs) using causal forest
cate_results <- 
  estimate_cates(
    Y, W, X, 
    ids = df_clean$id,
    weights = df_clean$wtssps
  )

# Display first few CATE estimates
cate_results$cates %>% head()
```

```
## # A tibble: 6 x 6
##      id treatment_effect standard_error ci_lower ci_upper
##   <dbl>            <dbl>          <dbl>    <dbl>    <dbl>
## 1     1             37.5           11.4    15.2      59.7
## 2     2             37.2           11.3    15.1      59.4
## 3     3             45.5           21.1     4.17     86.8
## 4     4             17.5           14.2   -10.2      45.3
## 5     5             35.9           28.9   -20.7      92.5
## 6     6             15.6           25.5   -34.4      65.5
## # i 1 more variable: quintile <int>
```

``` r
# Merge CATE estimates into main dataframe
df_clean <- 
  inner_join(
    df_clean,
    cate_results$cates,
    by = "id"
  )
```

The causal forest model consists of multiple decision trees that are specifically designed to estimate the effect of conservatism on Trump ratings, while accounting for confounding variables. Each tree estimates the treatment effect for individuals based on their characteristics, and the final CATE for each individual is obtained by averaging the treatment effect estimates from all trees in the forest. It is possible to visualize any of the individual trees from the causal forest to understand how the model makes its predictions.


``` r
# Visualize a random tree
tree_num <- sample(1:2000, 1)
plot(get_tree(cate_results$causal_forest, tree_num))
```

![](paper_files/figure-latex/visualize-tree-1.pdf)<!-- --> 


Given that we have individual level effects, we can also report the average treatment effect (ATE) along with its 95% confidence interval, in line with traditional approaches.


``` r
# Report ATE with 95% CI
ate_val <- round(cate_results$ate, 2)
ate_ci <- round(1.96 * cate_results$ate_se, 2)
print(paste("ATE:", ate_val, "+/-", ate_ci))
```

```
## [1] "ATE: 29.95 +/- 3.36"
```


We can also visualize the distribution of individual treatment effects using histograms and violin plots, stratified by quintiles of effect size. The default number of quantiles is 5 (quintiles), but this can be adjusted as needed. We use the `plot_treatment_effect_histogram` and `plot_treatment_effect_violin` functions from the `mediaeffects` package to create these visualizations.


``` r
# Visualize distribution by quintile
plot_treatment_effect_histogram(
  df_clean, 
  ate = cate_results$ate
)
```

![](paper_files/figure-latex/plot-histogram-1.pdf)<!-- --> 



``` r
# Show distribution within each quintile
plot_treatment_effect_violin(
  df_clean, 
  ate = cate_results$ate
)
```

![](paper_files/figure-latex/plot-violin-1.pdf)<!-- --> 


### Step 3: Which covariates drive heterogeneity?

Now that we have established that there is significant heterogeneity in the effect of conservatism on Trump ratings, and we have estimated individual-level treatment effects, we can explore which covariates are driving this heterogeneity. The `identify_moderators` function from the `mediaeffects` package helps us identify the most important moderators by calculating variable importance scores based on how much each covariate contributes to the heterogeneity in treatment effects.


``` r
# Identify which covariates drive heterogeneity
moderators <- identify_moderators(
  cate_results$causal_forest, 
  X
)

moderators$variable_importance
```

```
## # A tibble: 31 x 3
##    variable importance  rank
##    <chr>         <dbl> <int>
##  1 lkelyvot     0.313      1
##  2 income16     0.190      2
##  3 educ         0.120      3
##  4 prestg10     0.0672     4
##  5 age          0.0668     5
##  6 white        0.0354     6
##  7 compuse      0.0293     7
##  8 attend       0.0193     8
##  9 born         0.0184     9
## 10 pray         0.0173    10
## # i 21 more rows
```


We can visualize the top moderators using a heatmap, which shows how the estimated treatment effects vary across different levels of the top moderators. We use the `plot_moderator_heatmap` function from the `mediaeffects` package to create this visualization.


``` r
# Visualize top moderators across quintiles
plot_moderator_heatmap(
  df_clean, 
  moderators$top_variables
)
```

![](paper_files/figure-latex/plot-heatmap-1.pdf)<!-- --> 


Likelihood to vote seems like a strong moderator of this effect in this example. This suggests that among conservatives, those who are more likely to vote in the 2024 elections tend to have higher ratings of Donald Trump compared to those who are less likely to vote. This finding highlights the importance of voter engagement in shaping political attitudes within ideological groups.


## Verification phase

Now that we have identified likely moderators of heterogeneity using causal forests, we can verify these findings using traditional subgroup analysis as well as with interaction terms in linear regression models. This helps to ensure that the results are robust and interpretable in a linear regression framework.

First, we can look at subgroups by plotting Trump ratings by likelihood to vote among conservatives. We specifically focus on conservatives (treatment group) here to see how Trump ratings vary by likelihood to vote. We see that the subgroups who are more likely to vote tend to have higher Trump ratings.

![](paper_files/figure-latex/plot-subgroups-1.pdf)<!-- --> 

Second, we can fit a linear regression model with an interaction term between conservatism and likelihood to vote. This allows us to formally test whether the effect of conservatism on Trump ratings varies by likelihood to vote.


\begin{table}
\begin{center}
\begin{tabular}{l c}
\hline
 & Model 1 \\
\hline
(Intercept)           & $48.692^{***}$ \\
                      & $(2.723)$      \\
conservative          & $-18.418^{*}$  \\
                      & $(7.832)$      \\
lkelyvot              & $-4.632^{***}$ \\
                      & $(0.641)$      \\
conservative:lkelyvot & $12.401^{***}$ \\
                      & $(1.675)$      \\
\hline
R$^2$                 & $0.258$        \\
Adj. R$^2$            & $0.257$        \\
Num. obs.             & $3309$         \\
RMSE                  & $30.746$       \\
\hline
\multicolumn{2}{l}{\scriptsize{$^{***}p<0.001$; $^{**}p<0.01$; $^{*}p<0.05$}}
\end{tabular}
\caption{Statistical models}
\label{table:coefficients}
\end{center}
\end{table}

The results show that the interaction terms between conservatism and likelihood to vote are statistically significant, indicating that the effect of conservatism on Trump ratings does indeed vary by likelihood to vote. Specifically, the positive coefficients for the interaction terms suggest that as individuals report being more likely to vote, the positive effect of conservatism on Trump ratings increases. This finding aligns with the results from the causal forest analysis, providing further evidence that likelihood to vote is an important moderator of the relationship between conservatism and support for Donald Trump.

# Discussion

The use of causal forests provides a powerful and flexible approach to exploring heterogeneity in treatment effects, overcoming many of the limitations associated with traditional methods. By allowing for the automatic identification of complex interactions and nonlinear relationships, causal forests enable researchers to uncover nuanced patterns of heterogeneity that may be missed by traditional approaches. The illustrative example using the 2024 GSS data demonstrates how causal forests can be applied to estimate individual-level treatment effects and identify key moderators driving heterogeneity.

# References {-}






