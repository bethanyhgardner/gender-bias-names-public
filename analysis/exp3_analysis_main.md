Experiment 3: Main Analyses
================
2023-03-26

- <a href="#setup" id="toc-setup">Setup</a>
- <a href="#data-summary" id="toc-data-summary">Data Summary</a>
- <a href="#model-1-with-other-responses"
  id="toc-model-1-with-other-responses">Model 1: With <em>Other</em>
  Responses</a>
  - <a href="#odds-ratios-intercept" id="toc-odds-ratios-intercept">Odds
    Ratios: Intercept</a>
  - <a href="#odds-ratios-last-vs-firstfull"
    id="toc-odds-ratios-last-vs-firstfull">Odds Ratios: Last vs
    First+Full</a>
  - <a href="#odds-ratios-last-only" id="toc-odds-ratios-last-only">Odds
    Ratios: Last Only</a>
  - <a href="#odds-ratios-first-and-full-only"
    id="toc-odds-ratios-first-and-full-only">Odds Ratios: First and Full
    Only</a>
- <a href="#model-2-without-other-responses"
  id="toc-model-2-without-other-responses">Model 2: Without <em>Other</em>
  Responses</a>
  - <a href="#odds-ratios-intercept-1" id="toc-odds-ratios-intercept-1">Odds
    Ratios: Intercept</a>
  - <a href="#odds-ratios-last-vs-firstfull-1"
    id="toc-odds-ratios-last-vs-firstfull-1">Odds Ratios: Last vs
    First+Full</a>
  - <a href="#odds-ratios-last-only-1" id="toc-odds-ratios-last-only-1">Odds
    Ratios: Last Only</a>
  - <a href="#odds-ratios-first-and-full-only-1"
    id="toc-odds-ratios-first-and-full-only-1">Odds Ratios: First and Full
    Only</a>

# Setup

Variable names:

- Experiment: exp3\_
- Data (\_d\_)
  - d = main df
  - count = sums of response types
  - noOther = just *he* and *she* responses
- Models (\_m\_)
  - all = effect of Condition and Name Gender Rating, including *other*
    responses
  - cond = effect of Condition only
  - noOther = effect of Conditions (Last vs First+Full) and Name Gender
    Rating, only on *he* and *she* responses
  - FF = dummy coded with First + Full Name conditions as 0, Last Name
    condition as 1
  - L = dummy coded with Last Name condition as 0, First + Full Name
    conditions as 1

Load data and select columns used in model. See data/exp3_data_about.txt
for more details.

``` r
exp3_d <- read.csv("../data/exp3_data.csv",
                   stringsAsFactors = TRUE) %>%
  rename("Participant" = "SubjID", "Item" = "Name") %>%
  select(
    Participant, Condition, GenderRating,
    Item, He, She, Other
  )
str(exp3_d)
```

    ## 'data.frame':    8904 obs. of  7 variables:
    ##  $ Participant : Factor w/ 1272 levels "Exp3_P1","Exp3_P10",..: 974 974 974 974 974 974 974 330 330 330 ...
    ##  $ Condition   : Factor w/ 3 levels "first","full",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ GenderRating: num  5.22 1.24 5.86 3.75 6.78 4.34 2.41 6.24 2.61 6.82 ...
    ##  $ Item        : Factor w/ 63 levels "Ashley Cook",..: 6 9 13 43 47 52 62 2 16 20 ...
    ##  $ He          : int  0 1 0 0 0 0 1 0 1 0 ...
    ##  $ She         : int  0 0 1 0 1 1 0 0 0 1 ...
    ##  $ Other       : int  1 0 0 1 0 0 0 1 0 0 ...

Center gender rating for names: Original scale from 1 to 7, with 1 as
most masculine and 7 as most feminine. Mean-centered with higher still
as more feminine.

``` r
exp3_d %<>% mutate(GenderRatingCentered = scale(GenderRating, scale = FALSE))
```

Set contrasts for name conditions, now weighted to account for uneven
sample sizes. This uses Scott Fraundorf’s function for weighted
contrasts. (The psycholing package version doesn’t support doing 2v1
comparisons, only 1v1.) Condition1 is Last vs First+Full. Condition2 is
First vs Full.

``` r
source("centerfactor.R")
contrasts(exp3_d$Condition) <- centerfactor(
  exp3_d$Condition, c("last", "first")
)
contrasts(exp3_d$Condition)
```

    ##             [,1]        [,2]
    ## first  0.4009434 -0.48113208
    ## full   0.4009434  0.51886792
    ## last  -0.5990566  0.01886792

# Data Summary

Responses by condition.

``` r
exp3_d %<>% mutate(ResponseAll = case_when(
  He    == 1 ~ "He",
  She   == 1 ~ "She",
  Other == 1 ~ "Other"
))

exp3_d_count <- exp3_d %>%
  group_by(Condition, ResponseAll) %>%
  summarise(n = n()) %>%
  pivot_wider(
    names_from = ResponseAll,
    values_from = n
  ) %>%
  mutate(
    She_HeOther = She / (He + Other),
    She_He = She / He
  ) %>%
  select(She, He, Other, She_HeOther, She_He)
```

    ## Adding missing grouping variables: `Condition`

``` r
kable(exp3_d_count, digits = 3)
```

<table>
<thead>
<tr>
<th style="text-align:left;">
Condition
</th>
<th style="text-align:right;">
She
</th>
<th style="text-align:right;">
He
</th>
<th style="text-align:right;">
Other
</th>
<th style="text-align:right;">
She_HeOther
</th>
<th style="text-align:right;">
She_He
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
first
</td>
<td style="text-align:right;">
941
</td>
<td style="text-align:right;">
992
</td>
<td style="text-align:right;">
902
</td>
<td style="text-align:right;">
0.497
</td>
<td style="text-align:right;">
0.949
</td>
</tr>
<tr>
<td style="text-align:left;">
full
</td>
<td style="text-align:right;">
848
</td>
<td style="text-align:right;">
899
</td>
<td style="text-align:right;">
752
</td>
<td style="text-align:right;">
0.514
</td>
<td style="text-align:right;">
0.943
</td>
</tr>
<tr>
<td style="text-align:left;">
last
</td>
<td style="text-align:right;">
1079
</td>
<td style="text-align:right;">
1378
</td>
<td style="text-align:right;">
1113
</td>
<td style="text-align:right;">
0.433
</td>
<td style="text-align:right;">
0.783
</td>
</tr>
</tbody>
</table>

# Model 1: With *Other* Responses

Effects of Condition (first name, last name, full name) and Gender
Rating on the likelihood of a *she* response, as opposed to a *he* or
*other* response. Participant and Item are included as random
intercepts, with items defined as the unique first, last and first +
last name combinations. Because the condition manipulations were fully
between-subject and between-item, fitting a random slope model was not
possible.

Because Experiment 3 always introduces the character with a full name,
then manipulates the name form in the subsequent 3 references, the main
analysis is one model, as opposed to the 2 for Experiment 1.

Condition1 is the contrast between last and first+full. Condition2 is
the contrast between first and full.

``` r
exp3_m_all <- glmer(
  She ~ Condition * GenderRatingCentered + (1 | Participant) + (1 | Item),
  data = exp3_d, family = binomial
)
summary(exp3_m_all)
```

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: She ~ Condition * GenderRatingCentered + (1 | Participant) +  
    ##     (1 | Item)
    ##    Data: exp3_d
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   7825.8   7882.5  -3904.9   7809.8     8896 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -3.0250 -0.4836 -0.1394  0.5355  9.7282 
    ## 
    ## Random effects:
    ##  Groups      Name        Variance Std.Dev.
    ##  Participant (Intercept) 0.7931   0.8905  
    ##  Item        (Intercept) 0.4209   0.6488  
    ## Number of obs: 8904, groups:  Participant, 1272; Item, 63
    ## 
    ## Fixed effects:
    ##                                 Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)                     -1.52419    0.10101 -15.090   <2e-16 ***
    ## Condition1                       0.15326    0.09155   1.674   0.0941 .  
    ## Condition2                       0.09121    0.11595   0.787   0.4315    
    ## GenderRatingCentered             1.14845    0.06039  19.017   <2e-16 ***
    ## Condition1:GenderRatingCentered  0.10499    0.04875   2.153   0.0313 *  
    ## Condition2:GenderRatingCentered -0.05628    0.06294  -0.894   0.3712    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Correlation of Fixed Effects:
    ##             (Intr) Cndtn1 Cndtn2 GndrRC C1:GRC
    ## Condition1   0.000                            
    ## Condition2  -0.015  0.023                     
    ## GndrRtngCnt -0.287 -0.004  0.016              
    ## Cndtn1:GnRC -0.009 -0.495  0.000  0.025       
    ## Cndtn2:GnRC  0.016  0.000 -0.488 -0.023  0.009

- Fewer *she* responses overall

- Last Name vs First+Full Names condition effect only trending

- More *she* responses as first names become more feminine

- Larger effect of first name gender in First+Full Name conditions than
  in Last Name conditions, which makes sense because there are 4
  repetitions of the gendered first name, as opposed to only 1.

## Odds Ratios: Intercept

``` r
exp(get_intercept(exp3_m_all))
```

    ## [1] 0.2177969

``` r
exp(-get_intercept(exp3_m_all))
```

    ## [1] 4.591434

0.22x less likely to use *she* overall (or: 4.59x more likely to use
*he* and *other* overall), p\<.001

## Odds Ratios: Last vs First+Full

``` r
exp3_m_all %>%
  tidy() %>%
  filter(term == "Condition1") %>%
  pull(estimate) %>%
  exp()
```

    ## [1] 1.165628

1.17x more likely to use *she* than *he* and *other* in First + Full
compared to Last, p=0.09

## Odds Ratios: Last Only

Dummy code with Last Name as 0, so that intercept is the Last Name
condition only.

``` r
exp3_d %<>% mutate(Condition_Last = case_when(
  Condition == "first" ~ 1,
  Condition == "full"  ~ 1,
  Condition == "last"  ~ 0
))
exp3_d$Condition_Last %<>% as.factor()
```

Model with just Condition (to more directly compare to Exp 1).

``` r
exp3_m_cond_L <- glmer(
  She ~ Condition_Last + (1 | Participant) + (1 | Item),
  data = exp3_d, family = binomial
)
summary(exp3_m_cond_L)
```

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: She ~ Condition_Last + (1 | Participant) + (1 | Item)
    ##    Data: exp3_d
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   7962.3   7990.7  -3977.1   7954.3     8900 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -2.9131 -0.4946 -0.1440  0.5311  8.8113 
    ## 
    ## Random effects:
    ##  Groups      Name        Variance Std.Dev.
    ##  Participant (Intercept) 0.7738   0.8796  
    ##  Item        (Intercept) 5.3393   2.3107  
    ## Number of obs: 8904, groups:  Participant, 1272; Item, 63
    ## 
    ## Fixed effects:
    ##                 Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)     -1.74418    0.30138  -5.787 7.15e-09 ***
    ## Condition_Last1  0.24967    0.07807   3.198  0.00138 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Correlation of Fixed Effects:
    ##             (Intr)
    ## Condtn_Lst1 -0.160

``` r
exp(get_intercept(exp3_m_cond_L))
```

    ## [1] 0.1747886

``` r
exp(-get_intercept(exp3_m_cond_L))
```

    ## [1] 5.721196

0.17x times less likely to use *she* than *he* and *other* in the Last
Name condition (or: 5.72x more likely to use *he* and *other* in the
Last Name condition), p\<.001

## Odds Ratios: First and Full Only

Dummy code with First and Full Name as 0, so the intercept is the
combination of those two.

``` r
exp3_d %<>% mutate(Condition_FF = case_when(
  Condition == "first" ~ 0,
  Condition == "full"  ~ 0,
  Condition == "last"  ~ 1
))
exp3_d$Condition_FF %<>% as.factor()
```

Model with just Condition (to more directly compare to Exp 1).

``` r
exp3_m_cond_FF <- glmer(
  She ~ Condition_FF + (1 | Participant) + (1 | Item),
  data = exp3_d, family = binomial
)
summary(exp3_m_cond_FF)
```

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: She ~ Condition_FF + (1 | Participant) + (1 | Item)
    ##    Data: exp3_d
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   7962.3   7990.7  -3977.1   7954.3     8900 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -2.9131 -0.4946 -0.1440  0.5311  8.8113 
    ## 
    ## Random effects:
    ##  Groups      Name        Variance Std.Dev.
    ##  Participant (Intercept) 0.7738   0.8796  
    ##  Item        (Intercept) 5.3393   2.3107  
    ## Number of obs: 8904, groups:  Participant, 1272; Item, 63
    ## 
    ## Fixed effects:
    ##               Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)   -1.49451    0.29899  -4.998 5.78e-07 ***
    ## Condition_FF1 -0.24967    0.07807  -3.198  0.00138 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Correlation of Fixed Effects:
    ##             (Intr)
    ## Conditn_FF1 -0.100

``` r
exp(get_intercept(exp3_m_cond_FF))
```

    ## [1] 0.2243578

``` r
exp(-get_intercept(exp3_m_cond_FF))
```

    ## [1] 4.457166

0.22x times less likely to use *she* than *he* and *other* in the First
and Full Name conditions (or: 4.46x more likely to use *he* and *other*
in the First and Full Name conditions), p\<.001

# Model 2: Without *Other* Responses

The sentence completion prompt for Experiment 3 is more open-ended than
in Experiment 1. So, we get a much higher proportion of *other*
responses (31% vs 7%), which I didn’t anticipate.

``` r
sum(exp3_d$Other)
```

    ## [1] 2767

``` r
sum(exp3_d$Other) / length(exp3_d$Other)
```

    ## [1] 0.3107592

``` r
exp3_d_noOther <- exp3_d %>% filter(Other == 0)
```

So, rerun the main model predicting the likelihood of *she* responses vs
*he* responses, with *other* responses excluded.

``` r
exp3_m_noOther <- glmer(
  She ~ Condition * GenderRatingCentered + (1 | Participant) + (1 | Item),
  data = exp3_d_noOther, family = binomial
)
summary(exp3_m_noOther)
```

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: She ~ Condition * GenderRatingCentered + (1 | Participant) +  
    ##     (1 | Item)
    ##    Data: exp3_d_noOther
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   4209.0   4262.8  -2096.5   4193.0     6129 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -9.0294 -0.3424 -0.0521  0.2952 12.5650 
    ## 
    ## Random effects:
    ##  Groups      Name        Variance Std.Dev.
    ##  Participant (Intercept) 0.5394   0.7345  
    ##  Item        (Intercept) 0.6807   0.8250  
    ## Number of obs: 6137, groups:  Participant, 1223; Item, 63
    ## 
    ## Fixed effects:
    ##                                 Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)                     -0.42369    0.12377  -3.423 0.000618 ***
    ## Condition1                       0.25701    0.09784   2.627 0.008619 ** 
    ## Condition2                      -0.01457    0.12816  -0.114 0.909458    
    ## GenderRatingCentered             1.67709    0.08371  20.034  < 2e-16 ***
    ## Condition1:GenderRatingCentered  0.41954    0.07691   5.455  4.9e-08 ***
    ## Condition2:GenderRatingCentered -0.14909    0.11205  -1.331 0.183345    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Correlation of Fixed Effects:
    ##             (Intr) Cndtn1 Cndtn2 GndrRC C1:GRC
    ## Condition1   0.053                            
    ## Condition2  -0.020  0.005                     
    ## GndrRtngCnt -0.155 -0.005  0.005              
    ## Cndtn1:GnRC -0.007 -0.210  0.003  0.201       
    ## Cndtn2:GnRC  0.005  0.004 -0.182 -0.061 -0.053

These results are more similar to what we predicted from the previous
experiments:

- Fewer *she* responses overall
- Fewer *she* responses in the Last Name condition as compared to the
  First + Full Name conditions (although we wouldn’t predict as large as
  a difference as in Exp1, because here there is one instance of the
  first name in the Last Name condition)
- More *she* responses as first names become more feminine
- Larger effect of first name gender in First+Full Name conditions than
  in Last Name conditions (which makes sense because there are
  4repetitions of the gendered first name, as opposed to only 1.)

But, to keep the analyses consistent between experiments and avoid
post-hoc decision weirdness, both versions are reported.

## Odds Ratios: Intercept

``` r
exp(get_intercept(exp3_m_noOther))
```

    ## [1] 0.6546237

``` r
exp(-get_intercept(exp3_m_noOther))
```

    ## [1] 1.527595

0.65x less likely to use *she* than *he* overall (or: 1.53x more likely
to use *he* than *she* overall), p\<.001

## Odds Ratios: Last vs First+Full

``` r
exp3_m_noOther %>%
  tidy() %>%
  filter(term == "Condition1") %>%
  pull(estimate) %>%
  exp()
```

    ## [1] 1.293063

1.29x more likely to use *she* than *he* in First+Full than in Last (or:
1.29x more likely to use *he* than *she* in Last than in First+Full),
p\<.001

## Odds Ratios: Last Only

Dummy code with Last Name as 0, so that intercept is the Last Name
condition only.

``` r
exp3_d_noOther %<>% mutate(Condition_Last = case_when(
  Condition == "first" ~ 1,
  Condition == "full"  ~ 1,
  Condition == "last"  ~ 0
))
exp3_d_noOther$Condition_Last %<>% as.factor()
```

``` r
exp3_m_noOther_L <- glmer(
  She ~ Condition_Last + (1 | Participant) + (1 | Item),
  data = exp3_d_noOther, family = binomial
)
summary(exp3_m_noOther_L)
```

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: She ~ Condition_Last + (1 | Participant) + (1 | Item)
    ##    Data: exp3_d_noOther
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   4383.5   4410.4  -2187.8   4375.5     6133 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -7.4256 -0.3377 -0.0653  0.2875 10.2132 
    ## 
    ## Random effects:
    ##  Groups      Name        Variance Std.Dev.
    ##  Participant (Intercept)  0.4906  0.7004  
    ##  Item        (Intercept) 10.1950  3.1930  
    ## Number of obs: 6137, groups:  Participant, 1223; Item, 63
    ## 
    ## Fixed effects:
    ##                 Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)     -0.67726    0.41223  -1.643      0.1    
    ## Condition_Last1  0.37418    0.09174   4.079 4.53e-05 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Correlation of Fixed Effects:
    ##             (Intr)
    ## Condtn_Lst1 -0.135

``` r
exp(get_intercept(exp3_m_noOther_L))
```

    ## [1] 0.5080049

``` r
exp(-get_intercept(exp3_m_noOther_L))
```

    ## [1] 1.968485

0.51x times less likely to use *she* than *he* in the Last Name
condition (or: 1.97x more likely to use *he* than *she* in the Last Name
condition), p=.10

## Odds Ratios: First and Full Only

Dummy code with First and Full Name as 0, so the intercept is the
combination of those two.

``` r
exp3_d_noOther %<>% mutate(Condition_FF = case_when(
  Condition == "first" ~ 0,
  Condition == "full"  ~ 0,
  Condition == "last"  ~ 1
))
exp3_d_noOther$Condition_FF %<>% as.factor()
```

``` r
exp3_m_noOther_FF <- glmer(
  She ~ Condition_FF + (1 | Participant) + (1 | Item),
  data = exp3_d_noOther, family = binomial
)
summary(exp3_m_noOther_FF)
```

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: She ~ Condition_FF + (1 | Participant) + (1 | Item)
    ##    Data: exp3_d_noOther
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   4383.5   4410.4  -2187.8   4375.5     6133 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -7.4256 -0.3377 -0.0653  0.2875 10.2132 
    ## 
    ## Random effects:
    ##  Groups      Name        Variance Std.Dev.
    ##  Participant (Intercept)  0.4906  0.7004  
    ##  Item        (Intercept) 10.1949  3.1929  
    ## Number of obs: 6137, groups:  Participant, 1223; Item, 63
    ## 
    ## Fixed effects:
    ##               Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)   -0.30308    0.41002  -0.739     0.46    
    ## Condition_FF1 -0.37418    0.09174  -4.079 4.53e-05 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Correlation of Fixed Effects:
    ##             (Intr)
    ## Conditn_FF1 -0.088

``` r
exp(get_intercept(exp3_m_noOther_FF))
```

    ## [1] 0.7385373

``` r
exp(-get_intercept(exp3_m_noOther_FF))
```

    ## [1] 1.354028

0.74x times less likely to use *she* than *he* and *other* in the First
and Full Name conditions (or: 1.35x more likely to use *he* and *other*
in the First and Full Name conditions), p=.46
