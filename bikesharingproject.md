Bike sharing project
================
Dushyant Barot
2022-10-03

``` r
# read the data file
data = read.csv("bike_sharing_day.csv")
# load the packages required
library(ggplot2)          # Data visualization
library(readr)            # CSV file I/O, e.g. the read_csv function
library(base)             # for finding the file names
library(data.table)       # for data input and wrangling
library(dplyr)            # for data wranging
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:data.table':
    ## 
    ##     between, first, last

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(forecast)         # for time series forecasting
```

    ## Registered S3 method overwritten by 'quantmod':
    ##   method            from
    ##   as.zoo.data.frame zoo

``` r
library(tseries)          # for time series data    
library(lubridate)        # for date modifications
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following objects are masked from 'package:data.table':
    ## 
    ##     hour, isoweek, mday, minute, month, quarter, second, wday, week,
    ##     yday, year

    ## The following objects are masked from 'package:base':
    ## 
    ##     date, intersect, setdiff, union

``` r
library(tidyr)            # for data wrangling
library(magrittr)         # for data wrangling
```

    ## 
    ## Attaching package: 'magrittr'

    ## The following object is masked from 'package:tidyr':
    ## 
    ##     extract

``` r
library(knitr)
library(ggplot2) 
library(gridExtra)
```

    ## 
    ## Attaching package: 'gridExtra'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     combine

``` r
library(car)
```

    ## Loading required package: carData

    ## 
    ## Attaching package: 'car'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     recode

``` r
library(cowplot)
```

    ## 
    ## Attaching package: 'cowplot'

    ## The following object is masked from 'package:lubridate':
    ## 
    ##     stamp

Now let’s take a look at the distribution of count variable that we will
try to predict later on.

``` r
ggplot(data, aes(cnt)) + geom_histogram(color = "black", fill="darkblue", alpha = 0.8) +
  labs(x = "Bike rentals", y = "Count", title = "Distribution of bike rentals") +
  theme_minimal() + theme(panel.grid.major = element_line(color = "lightgrey"))
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](bikesharingproject_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

In some time periods very few bikes are rented also as the number of
bike rentals become large the corresponding time periods are small. One
factor that easily comes to mind that will affect number of bike rentals
is weather situation. In good weather we expect bike rentals to be high
and in bad weather we expect bike rentals to be low.

``` r
data %>% mutate(season = as.factor(season)) %>%
  ggplot(., aes(season, cnt)) + geom_boxplot(aes(color = season, fill = season), alpha = 0.4) +
  scale_x_discrete(labels = c("Spring", "Summer", "Fall", "Winter")) + 
  labs(x = "Season", y = "Number of rentals", title = "Number of rentals by season") +
  theme_minimal() + theme(legend.position = "none", panel.grid.major = element_line(color = "lightgrey"))
```

![](bikesharingproject_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

The hypothesis stated above does not hold as seen in the figure above.
The number of bike rentals in winter is almost the same as that in
summer and the number of bike rentals in spring is the lowest of all
seasons. What could be the reason behind this? One thing is clear that
bike rentals are not linearly related to weather situations. To
understand this we take a more direct look at the effect of weather on
bike rentals. For this we relabel the weather situations as good, fair,
bad, very bad.

``` r
data %>% mutate(weathersit = as.factor(weathersit)) %>%
  ggplot(., aes(weathersit, cnt)) + geom_boxplot(aes(color = weathersit, fill = weathersit), alpha = 0.4) +
  labs(x = "Weather", y = "Number of rentals") + ggtitle("Number of rentals by weather conditions") +
  scale_x_discrete(labels = c("Good", "Fair", "Bad", "Very bad")) +
  theme_minimal() + theme(panel.grid.major = element_line(color = "lightgrey"), 
                          legend.position = "none")
```

![](bikesharingproject_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
dummy <- data %>% group_by(as.factor(weathersit)) %>% 
  summarise(n = round(mean(cnt, na.rm = TRUE), digits = 0 ))
names(dummy) <- c("Weather", "Mean number of rentals")
dummy$Weather <- c("Good", "Fair", "Bad")

dummy %>% kable("html") 
```

<table>
<thead>
<tr>
<th style="text-align:left;">
Weather
</th>
<th style="text-align:right;">
Mean number of rentals
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Good
</td>
<td style="text-align:right;">
4877
</td>
</tr>
<tr>
<td style="text-align:left;">
Fair
</td>
<td style="text-align:right;">
4036
</td>
</tr>
<tr>
<td style="text-align:left;">
Bad
</td>
<td style="text-align:right;">
1803
</td>
</tr>
</tbody>
</table>

``` r
rm(dummy)
```

We see that weather has a clear influence on the count of bike rentals.
As the weather situation deteriorates the mean of bike rentals go down.
To get a deeper insight we will take a look at influence of specific
factors that affect weather situation, eg. temperature, windspeed and
air humidity.

``` r
p1 <- ggplot(data, aes(temp, cnt)) + geom_jitter(alpha = 0.4) +
  geom_smooth(formula = y ~ x, method = "lm", color = "red", se = FALSE, size = 1.2) +
  labs(x = "Temperature (in Celsius)", y = "Number of bike rentals") +
  ggtitle("Number of bike rentals by\ntemperature") +
  theme_minimal() + theme(panel.grid.major = element_line(color = "lightgrey"))

p2 <- ggplot(data, aes(atemp, cnt)) + geom_jitter(alpha = 0.4) + 
  geom_smooth(formula = y ~ x, method = "lm", color = "red", se = FALSE, size = 1.2) + 
  labs(x = "Temperature feel (in Celsius)", y = "") + 
  ggtitle("Number of bike rentals by\ntemperature feel") +
  theme_minimal() + theme(panel.grid.major = element_line(color = "lightgrey"))
plot_grid(p1, p2)
```

![](bikesharingproject_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

From the plots above we see that temperature and temperature feel are
two predictors that can be included in a linear model for bike rental.
But both should not be included as they are expected to be highly
correlated.

Windspeed is another factor that affects weather so let us now look at
how windspeed affects bike rentals.

``` r
plot(data$windspeed,data$cnt,xlab="wind speed",ylab="bike rentals",col="skyblue")
```

![](bikesharingproject_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

Higher the wind speed we see lower is the number of bike rentals. This
suggests that wind speed can also be used as a variable in a linear
model for bike rentals.

``` r
data %>% mutate(dteday = as.Date(dteday)) %>%
  ggplot(., aes(dteday, cnt)) + geom_point(color = data$cnt, alpha = 0.4) + 
  geom_smooth(formula = y ~ x,method = "lm", color = "red", se = FALSE) + 
  theme_minimal() + theme(panel.grid.major = element_line(color = "lightgrey")) + 
  labs(x = "Date", y = "Number of rentals") + 
  ggtitle("Number of bike rentals by date")
```

![](bikesharingproject_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->
We see that linear model is a good approximation of bike rentals in the
case of variation with date.

Now we will see variation with weekdays and holidays through box plots.

``` r
data <- data %>% mutate(workingday = as.factor(workingday), holiday = as.factor(holiday))

p1 <- ggplot(data, aes(workingday, cnt)) + 
  geom_boxplot(aes(color = workingday, fill = workingday), alpha = 0.4) +
  scale_x_discrete(labels = c("No", "Yes")) + theme_minimal() + 
  theme(legend.position = "none", panel.grid.major = element_line(color = "lightgrey")) + 
  labs(x = "Workday?", y = "Number of rentals", title = "Number of bike rentals by\nworkday/weekend")

p2 <- ggplot(data, aes(holiday, cnt)) + 
  geom_boxplot(aes(color = holiday, fill = holiday), alpha = 0.4) + 
  scale_x_discrete(labels = c("No", "Yes")) + theme_minimal() + 
  theme(legend.position = "none", panel.grid.major = element_line(color = "lightgrey")) + 
  labs(x = "Holiday?", y = "Number of rentals", title = "Number of bike rentals by\nholiday/not holiday") 

plot_grid(p1, p2)
```

![](bikesharingproject_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

With this we complete the EDA for the bike sharing dataset.

``` r
 cols(
   instant = col_double(),
   dteday = col_date(format = ""),
   season = col_double(),
   yr = col_double(),
   mnth = col_double(),
   holiday = col_double(),
   weekday = col_double(),
   workingday = col_double(),
   weathersit = col_double(),
   temp = col_double(),
   atemp = col_double(),
   hum = col_double(),
   windspeed = col_double(),
   casual = col_double(),
   registered = col_double(),
   cnt = col_double()
 )
```

    ## cols(
    ##   instant = col_double(),
    ##   dteday = col_date(format = ""),
    ##   season = col_double(),
    ##   yr = col_double(),
    ##   mnth = col_double(),
    ##   holiday = col_double(),
    ##   weekday = col_double(),
    ##   workingday = col_double(),
    ##   weathersit = col_double(),
    ##   temp = col_double(),
    ##   atemp = col_double(),
    ##   hum = col_double(),
    ##   windspeed = col_double(),
    ##   casual = col_double(),
    ##   registered = col_double(),
    ##   cnt = col_double()
    ## )

``` r
data <- data[,c("season", "workingday", "weathersit", "temp", "hum", "windspeed", "cnt")]
head(data)
```

    ##   season workingday weathersit     temp      hum windspeed  cnt
    ## 1      1          0          2 0.344167 0.805833 0.1604460  985
    ## 2      1          0          2 0.363478 0.696087 0.2485390  801
    ## 3      1          1          1 0.196364 0.437273 0.2483090 1349
    ## 4      1          1          1 0.200000 0.590435 0.1602960 1562
    ## 5      1          1          1 0.226957 0.436957 0.1869000 1600
    ## 6      1          1          1 0.204348 0.518261 0.0895652 1606

### Regression Analysis

Following is a linear model which fits the given data.

``` r
model <- lm(cnt ~ factor(season) + factor(workingday) + factor(weathersit) + temp + hum + windspeed, data = data)
summary(model)
```

    ## 
    ## Call:
    ## lm(formula = cnt ~ factor(season) + factor(workingday) + factor(weathersit) + 
    ##     temp + hum + windspeed, data = data)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -3714.8  -918.5  -243.6  1059.9  4214.4 
    ## 
    ## Coefficients:
    ##                     Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)           3024.4      349.3   8.659  < 2e-16 ***
    ## factor(season)2        932.3      178.6   5.220 2.34e-07 ***
    ## factor(season)3        483.2      235.6   2.051   0.0406 *  
    ## factor(season)4       1499.6      152.4   9.841  < 2e-16 ***
    ## factor(workingday)1    155.0      103.7   1.496   0.1352    
    ## factor(weathersit)2   -232.3      127.7  -1.818   0.0694 .  
    ## factor(weathersit)3  -1929.7      326.8  -5.905 5.43e-09 ***
    ## temp                  6159.1      481.1  12.802  < 2e-16 ***
    ## hum                  -2608.2      461.1  -5.656 2.23e-08 ***
    ## windspeed            -3306.0      674.5  -4.901 1.18e-06 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1296 on 721 degrees of freedom
    ## Multiple R-squared:  0.5577, Adjusted R-squared:  0.5522 
    ## F-statistic:   101 on 9 and 721 DF,  p-value: < 2.2e-16

``` r
anova(model)
```

    ## Analysis of Variance Table
    ## 
    ## Response: cnt
    ##                     Df     Sum Sq   Mean Sq  F value    Pr(>F)    
    ## factor(season)       3  950595868 316865289 188.5553 < 2.2e-16 ***
    ## factor(workingday)   1    5469478   5469478   3.2547   0.07164 .  
    ## factor(weathersit)   2  248464999 124232499  73.9263 < 2.2e-16 ***
    ## temp                 1  249905725 249905725 148.7100 < 2.2e-16 ***
    ## hum                  1   33093919  33093919  19.6930 1.052e-05 ***
    ## windspeed            1   40371975  40371975  24.0239 1.176e-06 ***
    ## Residuals          721 1211633428   1680490                       
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

The hypothesis tested for significance of regression:
$H_0:\beta_1=\beta_2=\cdots=\beta_k=0$ $H_A:\beta_j\neq 0$ for at least
one $j$.

The ANOVA table output divides the $MS_{reg}$ between the regressos. To
calculate the F-statistic one can either use the ANOVA table or use the
model summary. The reported F-statistic has value of 101 with a
$p$-value smaller than 0.05. Hence, we reject $H_0$.

The hypothesis for significance of individual coefficients:
$H_0:\beta_j=0$ $H_A:\beta_j\neq 0$ From the individual t-test
coefficients obtained from model summary we arrive at the conclusion
that workingday=1 and weathersit=1 are not statistically significant at
5% level. From the ANOVA table we come to the conclusion that adding the
variable workingday to the regression model is not statistically
significant so we will now leave it out of the model.

We now check the multicollinearity assumption.

``` r
vif(model)
```

    ##                        GVIF Df GVIF^(1/(2*Df))
    ## factor(season)     3.504848  3        1.232475
    ## factor(workingday) 1.010191  1        1.005082
    ## factor(weathersit) 1.780069  2        1.155072
    ## temp               3.369325  1        1.835572
    ## hum                1.873643  1        1.368811
    ## windspeed          1.186928  1        1.089462

All the values in the $\text{GVIF}^\text{1/(2*Df)}$ column are smaller
than 2 so we can conclude there is no multicollinearity present between
the predictor variables.

Now we draw some plots and run some tests to check the assumptions
behind the model.

``` r
  res.model = rstandard(model)
  par(mfrow=c(2,2))
  plot(model, which = c(1,1))
  hist(res.model,main="Histogram of standardised residuals")
  qqnorm(res.model,main="QQ plot of standardised residuals")
  qqline(res.model, col = 2, lty = 2)
  acf(res.model, main="ACF of standardised residuals")
```

![](bikesharingproject_files/figure-gfm/unnamed-chunk-16-1.png)<!-- -->

``` r
  print(shapiro.test(res.model))
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  res.model
    ## W = 0.98312, p-value = 1.845e-07

``` r
  print(durbinWatsonTest(model))
```

    ##  lag Autocorrelation D-W Statistic p-value
    ##    1       0.7807316     0.4369705       0
    ##  Alternative hypothesis: rho != 0

``` r
  print(ncvTest(model))
```

    ## Non-constant Variance Score Test 
    ## Variance formula: ~ fitted.values 
    ## Chisquare = 32.65956, Df = 1, p = 1.098e-08

From the diagnostic checks in Figure above and test we conclude that
error are not distributed normally. That is the assumption that errors
are random is violated. The ACF plot show that the autocorrelation
structure in the data is not captured by the model. From the residual
analysis we conclude that the model is not a good fit for the data.

We use backward elimination method to come up with a smaller linear
model.

``` r
drop1(model, test="F")
```

    ## Single term deletions
    ## 
    ## Model:
    ## cnt ~ factor(season) + factor(workingday) + factor(weathersit) + 
    ##     temp + hum + windspeed
    ##                    Df Sum of Sq        RSS   AIC  F value    Pr(>F)    
    ## <none>                          1211633428 10488                       
    ## factor(season)      3 208219020 1419852448 10598  41.3012 < 2.2e-16 ***
    ## factor(workingday)  1   3759531 1215392958 10489   2.2372    0.1352    
    ## factor(weathersit)  2  58714379 1270347807 10519  17.4694 3.901e-08 ***
    ## temp                1 275398673 1487032100 10636 163.8800 < 2.2e-16 ***
    ## hum                 1  53768096 1265401523 10518  31.9955 2.229e-08 ***
    ## windspeed           1  40371975 1252005403 10510  24.0239 1.176e-06 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

``` r
drop1(update(model, ~ . -factor(workingday)), test = "F")
```

    ## Single term deletions
    ## 
    ## Model:
    ## cnt ~ factor(season) + factor(weathersit) + temp + hum + windspeed
    ##                    Df Sum of Sq        RSS   AIC F value    Pr(>F)    
    ## <none>                          1215392958 10489                      
    ## factor(season)      3 208414000 1423806958 10598  41.269 < 2.2e-16 ***
    ## factor(weathersit)  2  57331974 1272724932 10518  17.029 5.937e-08 ***
    ## temp                1 280714291 1496107250 10639 166.757 < 2.2e-16 ***
    ## hum                 1  54968383 1270361341 10519  32.654 1.611e-08 ***
    ## windspeed           1  41004780 1256397738 10511  24.359 9.937e-07 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

With a reported p-value larger than 0.05 the partial F test suggests
that workingday variable can be left out of the model.

We fit a new model with variables weathersit, temp, hum and windspeed.

``` r
submodel <- lm(cnt ~ factor(season) + factor(weathersit) + temp + hum + windspeed, data = data)
summary(submodel)
```

    ## 
    ## Call:
    ## lm(formula = cnt ~ factor(season) + factor(weathersit) + temp + 
    ##     hum + windspeed, data = data)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -3830.0  -926.4  -253.8  1097.0  4095.7 
    ## 
    ## Coefficients:
    ##                     Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)           3128.9      342.5   9.135  < 2e-16 ***
    ## factor(season)2        927.0      178.7   5.187 2.78e-07 ***
    ## factor(season)3        471.3      235.7   2.000   0.0459 *  
    ## factor(season)4       1496.8      152.5   9.814  < 2e-16 ***
    ## factor(weathersit)2   -218.8      127.5  -1.715   0.0867 .  
    ## factor(weathersit)3  -1902.9      326.6  -5.827 8.51e-09 ***
    ## temp                  6205.4      480.5  12.913  < 2e-16 ***
    ## hum                  -2635.2      461.1  -5.714 1.61e-08 ***
    ## windspeed            -3330.8      674.9  -4.935 9.94e-07 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1297 on 722 degrees of freedom
    ## Multiple R-squared:  0.5564, Adjusted R-squared:  0.5514 
    ## F-statistic: 113.2 on 8 and 722 DF,  p-value: < 2.2e-16

``` r
anova(submodel)
```

    ## Analysis of Variance Table
    ## 
    ## Response: cnt
    ##                     Df     Sum Sq   Mean Sq F value    Pr(>F)    
    ## factor(season)       3  950595868 316865289 188.233 < 2.2e-16 ***
    ## factor(weathersit)   2  243512285 121756142  72.329 < 2.2e-16 ***
    ## temp                 1  255086767 255086767 151.533 < 2.2e-16 ***
    ## hum                  1   33942733  33942733  20.164 8.274e-06 ***
    ## windspeed            1   41004780  41004780  24.359 9.937e-07 ***
    ## Residuals          722 1215392958   1683370                      
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

``` r
res.model = rstandard(model)
  par(mfrow=c(2,2))
  plot(model, which = c(1,1))
  hist(res.model,main="Histogram of standardised residuals")
  qqnorm(res.model,main="QQ plot of standardised residuals")
  qqline(res.model, col = 2, lty = 2)
  acf(res.model, main="ACF of standardised residuals")
```

![](bikesharingproject_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->

``` r
  print(shapiro.test(res.model))
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  res.model
    ## W = 0.98312, p-value = 1.845e-07

``` r
  print(durbinWatsonTest(model))
```

    ##  lag Autocorrelation D-W Statistic p-value
    ##    1       0.7807316     0.4369705       0
    ##  Alternative hypothesis: rho != 0

``` r
  print(ncvTest(model))
```

    ## Non-constant Variance Score Test 
    ## Variance formula: ~ fitted.values 
    ## Chisquare = 32.65956, Df = 1, p = 1.098e-08

From the figure above and the test reports we come to the conclusion
that assumption of normality and randomness of errors is violated. The
ACF test and the D-W test indicate that the errors are highly correlated
so the new model is not good enough to capture the high correlation
between the predictor variables.

``` r
model.final<-glm(cnt ~ factor(season) + factor(workingday) + factor(weathersit) + temp + hum + windspeed, family=poisson(link="log"), data = data)
summary.glm(model.final)
```

    ## 
    ## Call:
    ## glm(formula = cnt ~ factor(season) + factor(workingday) + factor(weathersit) + 
    ##     temp + hum + windspeed, family = poisson(link = "log"), data = data)
    ## 
    ## Deviance Residuals: 
    ##     Min       1Q   Median       3Q      Max  
    ## -59.115  -15.771   -3.352   15.691   68.686  
    ## 
    ## Coefficients:
    ##                      Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)          7.933928   0.004272 1856.98   <2e-16 ***
    ## factor(season)2      0.313370   0.002270  138.02   <2e-16 ***
    ## factor(season)3      0.197897   0.002831   69.89   <2e-16 ***
    ## factor(season)4      0.458829   0.001978  231.93   <2e-16 ***
    ## factor(workingday)1  0.039978   0.001203   33.23   <2e-16 ***
    ## factor(weathersit)2 -0.055284   0.001488  -37.14   <2e-16 ***
    ## factor(weathersit)3 -0.745276   0.005472 -136.20   <2e-16 ***
    ## temp                 1.381391   0.005599  246.74   <2e-16 ***
    ## hum                 -0.572890   0.005445 -105.21   <2e-16 ***
    ## windspeed           -0.723194   0.008116  -89.10   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for poisson family taken to be 1)
    ## 
    ##     Null deviance: 668801  on 730  degrees of freedom
    ## Residual deviance: 298057  on 721  degrees of freedom
    ## AIC: 305476
    ## 
    ## Number of Fisher Scoring iterations: 4

``` r
deviance(model.final)
```

    ## [1] 298057.5

``` r
pchisq(model.final$deviance, df=model.final$df.residual, lower.tail=FALSE)
```

    ## [1] 0

The summary of the Poisson model indicates that all the predictor
variables have a statistically significant influence on the response
with all p-values \< 0.05. The hypothesis for testing the model: $H_0$:
the model fits the data well $H_A$: the model does not fit the data well

The p-value of 0 with chi-squar test statistic of 298057 and 721 degrees
of freedom suggests we should reject the $H_0$ hypothesis. Hence, we can
infer that the Poisson model is not adequate for the data.

``` r
anova(model.final,test="Chi")
```

    ## Analysis of Deviance Table
    ## 
    ## Model: poisson, link: log
    ## 
    ## Response: cnt
    ## 
    ## Terms added sequentially (first to last)
    ## 
    ## 
    ##                    Df Deviance Resid. Df Resid. Dev  Pr(>Chi)    
    ## NULL                                 730     668801              
    ## factor(season)      3   232841       727     435960 < 2.2e-16 ***
    ## factor(workingday)  1     1227       726     434732 < 2.2e-16 ***
    ## factor(weathersit)  2    65261       724     369472 < 2.2e-16 ***
    ## temp                1    56481       723     312990 < 2.2e-16 ***
    ## hum                 1     6916       722     306074 < 2.2e-16 ***
    ## windspeed           1     8017       721     298057 < 2.2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

``` r
drop1(model.final, test="Chi")
```

    ## Single term deletions
    ## 
    ## Model:
    ## cnt ~ factor(season) + factor(workingday) + factor(weathersit) + 
    ##     temp + hum + windspeed
    ##                    Df Deviance    AIC   LRT  Pr(>Chi)    
    ## <none>                  298057 305476                    
    ## factor(season)      3   367366 374779 69309 < 2.2e-16 ***
    ## factor(workingday)  1   299167 306584  1110 < 2.2e-16 ***
    ## factor(weathersit)  2   320701 328116 22644 < 2.2e-16 ***
    ## temp                1   359006 366422 60948 < 2.2e-16 ***
    ## hum                 1   309055 316472 10997 < 2.2e-16 ***
    ## windspeed           1   306074 313491  8017 < 2.2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

The backward elimination method using partial Chisquare test with
p-values less than 0.05 along with the ANOVA tables we infer that all
the predictor variables are significant and suggest that a smaller
linear model is not possible.

### Residual Analysis

``` r
par(mfrow=c(1,2))
plot(residuals(model.final, type="deviance"))
qqnorm(residuals(model.final, type="deviance"))
qqline(residuals(model.final, type="deviance"), lty = 2, col = 2)
```

![](bikesharingproject_files/figure-gfm/unnamed-chunk-27-1.png)<!-- -->

``` r
shapiro.test(residuals(model.final,type="deviance"))
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  residuals(model.final, type = "deviance")
    ## W = 0.98468, p-value = 6.256e-07

The figure above of the plots of residuals resulting from the Poisson
model. We observe that the errors are not spread randomly. S-W test
suggests that the normality does not hold. Hence, we can conclude that
the model is not adequate.

### Conclusion

The aim of the project was to develop a model using multiple linear
regression which can be used to predict number of bike rentals. The
predictor variables were highly correlated. We concluded that a multiple
linear regression model with predictor variables in the data is not
adequate for predicting bike rentals.
