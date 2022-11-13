BTC/ETH check
================

This file attemps to reconcile results from BTC and ETH groups for
granger causality

### Load and merge data

``` r
> library(dplyr)
> library(readr)
> library(lubridate)
> library(zoo)
> library (lmtest)
> #load data
>   T5ybe <- read_csv("Clean Data/5ybreakevenclean.csv")
>   gold <- read_csv("Clean Data/goldclean.csv")
>   gold_ETH <- read_csv("ETH_data/Gold.csv")
> #merge data
>   merged <- inner_join(gold,T5ybe,by=c('Date'='T5YIE'))
>   merged$Date=ymd(merged$Date) #convert to date
```

### Run team ETH results in python code

``` r
> library(reticulate)
> py_install("pandas")
> py_install("numpy")
> py_install("statsmodels")
> 
```

``` python
> import pandas as pd
+ import numpy as np 
+ from statsmodels.tsa.stattools import grangercausalitytests
+ 
+ #Gold 
+ df_gold = pd.read_csv('ETH_data/Gold.csv')
+ # GOLD
+ #inflation=['T5YIE', 'T10YIE', 'T5YIFR']
+ inflation=['T5YIE']
+ for element in inflation: 
+   result=grangercausalitytests(df_gold[['Response', element]], maxlag=[5])
+   print(inflation, 'response', result)

Granger Causality
number of lags (no zero) 5
ssr based F test:         F=17.5949 , p=0.0000  , df_denom=7184, df_num=5
ssr based chi2 test:   chi2=88.1093 , p=0.0000  , df=5
likelihood ratio test: chi2=87.5741 , p=0.0000  , df=5
parameter F test:         F=17.5949 , p=0.0000  , df_denom=7184, df_num=5
['T5YIE'] response {5: ({'ssr_ftest': (17.594913631820692, 2.295938548368447e-17, 7184.0, 5), 'ssr_chi2test': (88.10927309364551, 1.6763593914671895e-17, 5), 'lrtest': (87.57414857427648, 2.1711443542335234e-17, 5), 'params_ftest': (17.59491363182082, 2.295938548367566e-17, 7184.0, 5.0)}, [<statsmodels.regression.linear_model.RegressionResultsWrapper object at 0x000001AD6FEC5670>, <statsmodels.regression.linear_model.RegressionResultsWrapper object at 0x000001AD6FEC5B80>, array([[0., 0., 0., 0., 0., 1., 0., 0., 0., 0., 0.],
       [0., 0., 0., 0., 0., 0., 1., 0., 0., 0., 0.],
       [0., 0., 0., 0., 0., 0., 0., 1., 0., 0., 0.],
       [0., 0., 0., 0., 0., 0., 0., 0., 1., 0., 0.],
       [0., 0., 0., 0., 0., 0., 0., 0., 0., 1., 0.]])])}
```

### Check R results

``` r
> grangertest(Response ~ T5YIE, order = 5, data = gold_ETH) 
Granger causality test

Model 1: Response ~ Lags(Response, 1:5) + Lags(T5YIE, 1:5)
Model 2: Response ~ Lags(Response, 1:5)
  Res.Df Df      F    Pr(>F)    
1   7184                        
2   7189 -5 17.595 < 2.2e-16 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

We can see that the Python and R code results are **EXACTLY** the
samewhen you look at the `F score`. This suggests that the difference in
results comes from the data used and not due to any differences in the
functions.
