Analysis of bikesharing_day dataset.

Different models like Linear Regression, Decision Tree and Random Forest were used.

Using Variation Inflation Factor method features we were left with were: season, workingday, weathersit, temp, windspeed. Adjusted R2_score achieved was: 0.5043.

Using Backward elimination method features we were left with were: 'season', 'weathersit', 'temp', 'hum','windspeed'. Adjusted R2_score achieved was: 0.5246.

Using Forward Selection method features we were left with were: ['atemp', 'weathersit', 'season', 'windspeed', 'hum']. Adjusted R2_score achieved was: 0.5263.
