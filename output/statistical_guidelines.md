# Statistical Guidelines

# Topics
These statistical guidelines represent preferences for statistical inference used by the M&I team. The original list was created by one of the former M&I General Managers, [Peter Ellis](http://ellisp.github.io/) and extended by the M&I senior data scientists.

## Planning
1.	Think through the **purpose of inference**  
* prediction/forecasting,
* population estimation
* theoretical understanding

and adapt the modelling strategy appropriately.


## Data preparation
1.	Use **multiple imputation** to impute missing values. An example of using multivariate imputation by chained equations is shown below.

```r
library(mice)

# Perform multiple imputation for missing values in the dataset. This section takes ~50 minutes to execute.
# Create a predictor matrix for the imputation that excludes the ID column
predictor_matrix <- 1- diag(1, ncol(mothers))
predictor_matrix[, grep(idcol_mum, names(mothers)) ] <- 0
imputation <- mice( data = mothers, m = 5, predictorMatrix = predictor_matrix )
mothers_imputed <- mice::complete(imputation, "long", inc= FALSE)

# The mice function will give us m=5 datasets with imputed values in a long form meaning each snz_uid will appear m=5 times
# with .imp indicating which of the 5 datasets the observation belongs to so take the mean imputed value for numeric 
# variables, round them if they are integer and the most commonly imputed value for categorical variables
imputed_num <- mothers_imputed %>% 
  group_by(snz_uid) %>% 
  select(snz_uid, numcols_mum) %>% 
  summarise_all(funs(mean))
```
2.	**Bootstrap or cross-validation should encompass data processing** (e.g. imputation and other pre modelling tasks should ideally be repeated for each resample).

## Modelling
1.	Consider **robust methods** when appropriate to the question - e.g. trimmed means rather than means, robust regression rather than ordinary least squares. 
2.	Two-way cross tabulations can be useful for presentation and exploration, what is better is a **single general model** (additive and not saturated) that looks at the effects of various variables while controlling for the others.
3.	Confidence or credibility intervals for **effect sizes** are better than hypothesis tests when there is a choice.  
4.	Use **prediction intervals** when forecasting or constructing prediction individual results. These intervals are wider than confidence intervals for effects sizes since it takes into account the uncertainty in the population estimate and the data scatter.
5.	The **humble out of the box tests**, including simple Chi Square tests of independence between two variables in cross tabulations with counts, do have their place. Be careful when the sample sizes are very large - everything will probably look significant.
6. If it is important enough to justify the effort then correct for **multiple comparisons**. If the correction is not justified then keep in mind for interpretation. An example of Tukey's multiple comparison is shown below for R and SAS.

```r
# example of one way of doing a multiple comparison

anova_model <- aov(tgt ~ income_band + region, data = data_pop)
posthoc <- TukeyHSD(x=anova_model, 'income_band', conf.level=0.95)
```


```sas
/* tukey test for multiple comparisons on both main effects */
  
proc glm data = data_pop;
class income_band region;
model tgt = income_band region;
means income_band region / tukey;
run;
```
7.	**Cross-validation or bootstrap** validation should be used when assessing overall value, fit of important models or for choosing between predictive model types. Bootstrap for effect sizes when its important and / or there is doubt about the assumptions behind the standard estimates.
8.	**Avoid stepwise regression** as it can encounter problems with collinearity and produce coefficients with magnitudes that are biased upwards. Use lasso, ridge regression, or the generalised form of the two; elastic net regularization. This should help reduce the number of explanatory variables or to deal with collinearity. Use the `glmnet` function in R or the `glmselect` procedure in SAS.
9.	Complex surveys should generally be analysed with **survey-specific methods** such as the `survey` package in R or the `surveyselect`, `surveymeans`, `surveyfreq`, `surveylogistic`, `surveyreg`, `surveyphreg` procedures in SAS. A set of examples is shown below for R and SAS.

```r
library(survey)

# example of how to create a survey design object and feed it into a survey glm model
modeldatalag <- modeldata %>% group_by(uid) %>%
  mutate(total_unweighted_amount_lag1 = lag(total_unweighted_amount, 1), 
         total_unweighted_amount_lag2 = lag(total_unweighted_amount, 2) ) %>%
  filter(roi_period> -3);

# do not forget the replicate weights so many people do  
repweights <- as.data.frame(modeldatalag[, grepl("rep_wgt_adj", names(modeldatalag))]);
modeldatalag_svy <- svrepdesign(weights = ~sofie_weight_adj, 
                                repweights = repweights, 
                                data = select(modeldatalag,-uid), 
                                type="JK1", scale = 0.99);

# use a survey glm to get more reliable coefficient and error estimates    
weighted_logreg <- svyglm(non_zero~as.factor(roi_period) * k10_band, design=modeldatalag_svy, family=binomial(link="logit") );
```


```sas
/* example snippet from a macro creating weight adjusted cross tabulations in sas */
/* do not forget the replicate weights so many people do */
  
proc surveyfreq data=&table_in. 
   weight sofie_weight_adj; 
   repweight rep_wgt_adj_1-rep_wgt_adj_100 ;
	 tables &var1. * &var2. /  chisq cl clwt row col or plots=all;
run;
```
10.	Take **spatial correlation and time series autocorrelation** seriously and if present use the right methods to deal with them. Methods can be found in various packages such as`gam` and `mgcv`.
11.	Methods like clustering, factor analysis and topic modelling are **handy exploratory tools but do not reify the results**.
12.	Use multi-stage modelling methods like propensity score modelling, two stage least squares or structural equation modelling when needed. However, **do not slip into thinking they are a get out of jail card** on causality.
13.	Be alert to the need for **mixed effects models** for grouped data or repeated measurement. Analysis can be done using a package like `lme4`.

## Interpretation

1.	Always compare the results to "but **if this was just down to chance**, what might we see?".

Last updated Sep-2017 by EW




