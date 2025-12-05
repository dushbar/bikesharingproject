Analysis of Bike Sharing dataset using Linear Regression 

This repository contains linear regression analysis, feature selection using PCA, VIF, Subset Selection and application of Shrinkage Methods. 

It contains five Jupyter notebooks:

- **`EDA.ipynb`** — exploring trends/patterns in the data
- **`subset_selection.ipynb`** — implemented forward and backward selection
- **`Shrinkage_methods.ipynb`** — explored impact of Ridge and Lasso on coefficients under presence of multicollinearity.
- **`VIF.ipynb`** — feature selection using VIF
- **`PCA.ipynb`** — implemented principal components regression

---

## Table of Contents

- [Motivation](#motivation)  
- [Key Findings (Summary)](#key-findings-summary)  
- [Notebook Details](#notebook-details)  

---

## Motivation

Linear regression is explored through the bike sharing dataset, especially the effect of multicollinearity.
This project aims to:

- Explore patterns in the data through visualizations
- Explore the effects of multicollinearity on coefficients using Ridge regression, Lasso regression 
- Carry out feature selection using Lasso, PCA, VIF and subset selection

---

## Key Findings (Summary)

### **Presence of multicollinearity**
- Correlation heatmap shows moderate to high multicollinearity.
- This was confirmed by high VIF of some features.
- Multicollinearity was expected because features like `atemp,temp` are expected to have strong correlation.

### **Feature Selection/Extraction**
- Feature selection was performed using forward selection, backward selection, VIF and Lasso
- A smaller subset of principal components was used to fit a regression model.

---

## Notebook Details

### **EDA.ipynb**
- Load and clean the data
- Boxplots, histograms of count of bike rentals for different levels of categorical variables 
- Correlation heatmap


### **subset_selection.ipynb**
- Implement forward and backward selection
- Both AIC and Adjusted-$R^2$ were used to find the best model

### **Shrinkage_methods.ipynb**
- Effect of Lasso and Ridge regression on coefficients
- Feature selection using Lasso

### **VIF.ipynb**
- Threshold of VIF was set to remove features with VIF above this threshold

### **PCA.ipynb**
- Plotted the first two principal components
- Loading vectors for each features were also plotted
- Regression was done using the principal components.


---
