Granger test
================

### Load and merge data

``` r
> library(dplyr)
> library(readr)
> library(lubridate)
> #load data
>   T5ybe <- read_csv("Clean Data/5ybreakevenclean.csv")
>   gold <- read_csv("Clean Data/goldclean.csv")
> #merge data
>   merged <- inner_join(gold,T5ybe,by=c('Date'='T5YIE'))
>   merged$Date=ymd(merged$Date) #convert to date
```

### Calculate returns

``` r
> library(zoo)
> merged$gold_dr <- (merged$Price-lag(merged$Price)) / lag(merged$Price)
> merged$T5ybe_dr <- (merged$ifl-lag(merged$ifl)) / lag(merged$ifl)
```

### Checking the order of variable input

The following output puts **inflation** before **gold**

``` r
> library (lmtest)
> grangertest(T5ybe_dr ~ gold_dr, order = 1, data = merged) 
Granger causality test

Model 1: T5ybe_dr ~ Lags(T5ybe_dr, 1:1) + Lags(gold_dr, 1:1)
Model 2: T5ybe_dr ~ Lags(T5ybe_dr, 1:1)
  Res.Df Df      F Pr(>F)
1   3168                 
2   3169 -1 0.3681 0.5441
```

The following output puts **gold** before **inflation**

``` r
> grangertest(gold_dr ~ T5ybe_dr, order = 1, data = merged) 
Granger causality test

Model 1: gold_dr ~ Lags(gold_dr, 1:1) + Lags(T5ybe_dr, 1:1)
Model 2: gold_dr ~ Lags(gold_dr, 1:1)
  Res.Df Df      F    Pr(>F)    
1   3168                        
2   3169 -1 27.731 1.487e-07 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

The following code performs the granger test manually

``` r
> results<-lm(gold_dr ~ lag(gold_dr) + lag(T5ybe_dr),data=merged)
> summary(results)

Call:
lm(formula = gold_dr ~ lag(gold_dr) + lag(T5ybe_dr), data = merged)

Residuals:
      Min        1Q    Median        3Q       Max 
-0.051365 -0.002572 -0.000071  0.002563  0.051356 

Coefficients:
               Estimate Std. Error t value Pr(>|t|)    
(Intercept)   0.0001142  0.0001225   0.932    0.351    
lag(gold_dr)  0.0162270  0.0177236   0.916    0.360    
lag(T5ybe_dr) 0.0197226  0.0037452   5.266 1.49e-07 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Residual standard error: 0.006898 on 3168 degrees of freedom
  (2 observations deleted due to missingness)
Multiple R-squared:  0.009185,  Adjusted R-squared:  0.00856 
F-statistic: 14.68 on 2 and 3168 DF,  p-value: 4.486e-07
```

The results imply that you get the same resutls when you put the
variable you want to test **FIRST**

### Correct granger tests

``` r
>   grangertest(gold_dr ~ T5ybe_dr, order = 1, data = merged) 
Granger causality test

Model 1: gold_dr ~ Lags(gold_dr, 1:1) + Lags(T5ybe_dr, 1:1)
Model 2: gold_dr ~ Lags(gold_dr, 1:1)
  Res.Df Df      F    Pr(>F)    
1   3168                        
2   3169 -1 27.731 1.487e-07 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
>   grangertest(gold_dr ~ T5ybe_dr, order = 2, data = merged) 
Granger causality test

Model 1: gold_dr ~ Lags(gold_dr, 1:2) + Lags(T5ybe_dr, 1:2)
Model 2: gold_dr ~ Lags(gold_dr, 1:2)
  Res.Df Df      F    Pr(>F)    
1   3165                        
2   3167 -2 14.723 4.318e-07 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
>   grangertest(gold_dr ~ T5ybe_dr, order = 3, data = merged) 
Granger causality test

Model 1: gold_dr ~ Lags(gold_dr, 1:3) + Lags(T5ybe_dr, 1:3)
Model 2: gold_dr ~ Lags(gold_dr, 1:3)
  Res.Df Df      F    Pr(>F)    
1   3162                        
2   3165 -3 10.058 1.356e-06 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
>   grangertest(gold_dr ~ T5ybe_dr, order = 4, data = merged) 
Granger causality test

Model 1: gold_dr ~ Lags(gold_dr, 1:4) + Lags(T5ybe_dr, 1:4)
Model 2: gold_dr ~ Lags(gold_dr, 1:4)
  Res.Df Df      F    Pr(>F)    
1   3159                        
2   3163 -4 17.961 1.356e-14 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
>   grangertest(gold_dr ~ T5ybe_dr, order = 5, data = merged) 
Granger causality test

Model 1: gold_dr ~ Lags(gold_dr, 1:5) + Lags(T5ybe_dr, 1:5)
Model 2: gold_dr ~ Lags(gold_dr, 1:5)
  Res.Df Df      F    Pr(>F)    
1   3156                        
2   3161 -5 14.839 2.035e-14 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```
