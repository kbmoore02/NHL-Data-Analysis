---
layout: post
title: "Supervised Learning Methods"
author: Kelsey Moore
description: Apply supervised learning to address the primary research question
image: /assets/images/sup_header.jpg
---

# Primary Research Question

My primary research goal is to see if I can predict a BYU student's favorite genre of music based on factors such as their major, their gender, if they played sports in high school, their favorite movie genre, what streaming service they use the most, their favorite social media app, and what part of the country they want to settle in.

# K-Nearest Neighbors

I created two knn models to predict favorite music genre -- one with all of the variables in the dataset [hereafter Model 1], and one with just the variables specified above [hereafter Model 2] -- using the sklearn `KNeighborsClassifier()` package. In order to determine the best values for the hyperparameters `n_neighbors` and `weights`, I used `GridSearchCV()`. For both models, the best hyperparameter values were `n_neighbors=2` (out of 1,2,3,4,5) and `weights='uniform'` (out of 'uniform' and 'distance').

Both models have the same accuracy of 0.27727. The overall ROC AUC for Model 1 is 0.52061, while the ROC AUC for Model 2 is 0.51478, so Model 1 performs slightly better, but it's not really a significant difference. Here are the one-vs-rest ROC curves for each music genre:

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/knn_model1_roc.png" alt="" style="width:400px;">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/knn_model2_roc.png" alt="" style="width:400px;">
</p>

As we can see based on these graphs and metrics, these are not great models for predicting a BYU student's favorite music genre.

# Random Forest

I created two random forest models to predict favorite music genre -- one with all of the variabels in the dataset [hereafter Model 3], and one with just the variables specified above [hereafter Model 4] -- using the sklearn `RandomForestClassifier()` package. I did not perform any hyperparameter tuning for these models.

Both models have the same accuracy of 0.32273, which is slightly better than the KNN models. The overall ROC AUC for Model 3 is 0.58351, while the ROC AUC for Model 4 is 0.58976, so Model 4 performs slightly better, but again, it's not really a significant difference. Here are the one-vs-rest ROC curves for each music genre:

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/knn_model3_roc.png" alt="" style="width:400px;">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/knn_model4_roc.png" alt="" style="width:400px;">
</p>

As we can see based on these graphs and metrics, these models are slightly better than the KNN models, but they still are not great models for predicting a BYU student's favorite music genre.

## Feature Importance

The `RandomForestClassifer()` model allows us to look at feature importances. 

<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/feature_importances.png" alt="" style="width:600px;">

According to this, the top 7 most importance features for predicting a BYU student's favorite music genre are Salary, BYUFounded, College, YHowTall, YearInternet, HoursHomework, and Movie, only 2 of which are part of the 7 variables I selected for my original research question. I'm curious to see how a model with just these features performs.

## Logistic Regression

Using sklearn's `LogisticRegression()` function, I created a model to predict Music using Salary, BYUFounded, College, YHowTall, YearInternet, HoursHomework, and Movie, the features we selected above.

The model's accuracy is 0.32727, while its ROC AUC is 0.52589. Here are the one-vs-rest ROC curves for each music genre: 

<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/knn_model5_roc.png" alt="" style="width:400px;">

This is about the same as the KNN and RF models, so we still can't make very good predictions of a BYU student's favorite music genre.
