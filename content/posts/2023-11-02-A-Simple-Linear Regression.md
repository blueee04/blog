+++
title = "A Simple Linear Regression"
description = "Implementing a Simple Linear Regression "
date = "2023-11-02"
tags = ["Data Analysis","Machine Learning","Data Science"]
summary = "Implementing a Simple Linear Regression "
+++

### Introduction

Let's dive into the world of simple linear regression, a fundamental statistical technique used to model the relationship between two quantitative variables.

### The Problem

Our goal is to predict housing prices in California based on various factors such as location, median income, housing density, and more. By understanding the relationship between these variables and housing prices, we can provide valuable insights to stakeholders in the real estate industry.

[dataset](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.fetch_california_housing.html)


### Explore the Data

Perform exploratory data analysis (EDA) to gain insights into the dataset. Visualize key variables using scatter plots, histograms, and correlation matrices to understand their distributions and relationships.
![data](https://raw.githubusercontent.com/blueee04/blog/main/content/images/2023-04-02-A-Simple-Linear%20Regression/Screenshot%20from%202024-02-13%2004-07-56.png)
![data](https://raw.githubusercontent.com/blueee04/blog/main/content/images/2023-04-02-A-Simple-Linear%20Regression/Screenshot%20from%202024-02-13%2004-08-07.png)
![data](https://raw.githubusercontent.com/blueee04/blog/main/content/images/2023-04-02-A-Simple-Linear%20Regression/Screenshot%20from%202024-02-13%2004-08-16.png)

### Train the model

```py
regression = LinearRegression()
regression.fit(X_train,y_train)
```
Here we are fitting the model using Linear Regression

### Evaluate the model

We evaluate the performance of the model using metrics such as Mean Squared Error (MSE) and Ridge Regressor

```py
mse = cross_val_score(regression,X_train,y_train,scoring="neg_mean_squared_error", cv = 10)
```
```py
ridgecv = GridSearchCV(ridge_regressor,parameters,scoring = 'neg_mean_squared_error', cv = 5)
```

### Make Prediction
Now we predict the result using 

```py
reg_pred = regression.predict(X_test)
```
```py
ridge_pred = ridgecv.predict(X_test)
```

### Visualize the data

We visualize the data using seaborn, displt function
![graph](https://raw.githubusercontent.com/blueee04/blog/main/content/images/2023-04-02-A-Simple-Linear%20Regression/Screenshot%20from%202024-02-13%2004-08-42.png)
![graph](https://github.com/blueee04/blog/blob/main/content/images/2023-04-02-A-Simple-Linear%20Regression/Screenshot%20from%202024-02-13%2004-08-53.png)
![graph](https://raw.githubusercontent.com/blueee04/blog/main/content/images/2023-04-02-A-Simple-Linear%20Regression/Screenshot%20from%202024-02-13%2004-09-00.png)


