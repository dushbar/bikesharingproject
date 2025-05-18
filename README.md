# Bikesharing Data Analysis

This project analyzes daily bike-sharing data from Washington, D.C., to uncover patterns, trends, and factors affecting the number of rentals. The final notebook includes a full data science workflow, from exploratory data analysis to model building and evaluation.

## Dataset
Dataset from [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/Bike+Sharing+Dataset)

The dataset `bikesharing_day.csv` contains daily data from a bike-sharing system and includes:

- **Date & seasonality**: Year, month, weekday, season
- **Weather information**: Temperature, humidity, wind speed, weather condition
- **User types**: Casual, registered
- **Target**: Total number of rentals (`cnt`)

## Analysis Steps

- Initial data exploration and summary statistics
- Visualizations of rental trends across seasons, working days, and weather conditions
- Correlation analysis of features
- Model building using:
  - **Linear Regression (feature selection using VIF method)**
  - **Linear Regression Backward Selection**
  - **Linear Regression Forward Selection**
  - **Decision Tree Regressor**
  - **Random Forest Regressor**
 

## Model Performance

| Model                  | R² Score | Adjusted R² Score |
|-----------------------|----------|-------------------|
| Linear Regression (feature selection using VIF method)    | 0.521     | 0.504             |
| Linear Regression Backward Selection    | 0.5409     | 0.5246              |
| Linear Regression Forward Selection         | 0.5426     | 0.5263              |
| Decision Tree      | 0.2246     | 0.1797              |
| Random Forest | 0.5656     |   0.5404          | 

## Key Findings

- Bike rentals increase significantly in **summer and fall**.
- **Temperature** and **season** are positively correlated with rental count.
- **Humidity** and **bad weather** reduce rental volume.
- **Random Forest** tends to perform better than simple models like Linear Regression.
