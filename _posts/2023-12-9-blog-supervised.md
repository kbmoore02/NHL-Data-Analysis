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

I created two knn models to predict favorite music genre -- one with all of the variables in the dataset [hereafter Model 1], and one with just the variables specified above [hereafter Model 2] -- using the sklearn `KNeighborsClassifier()` package. In order to determine the best values for the hyperparameters `n_neighbors` and `weights`, I used `GridSearchCV()`. For model 1, the best hyperparameter values were `n_neighbors=10` (out of 1,2,3,4,5,6,7,8,9,10) and `weights='distance'` (out of 'uniform' and 'distance'), and for model 2, the best values were `n_neighbors=10` and `weights='uniform'`.

Model 1 has an accuracy of 0.33636, and model 2 has an accuracy of 0.3. The overall ROC AUC for model 1 is 0.58656, while the ROC AUC for model 2 is 0.59283, so model 2 performs slightly better, but this doesn't seem like a very significant difference. Here are the one-vs-rest ROC curves for each music genre:

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/knn_model1_roc.png" alt="" style="width:400px;">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/knn_model2_roc.png" alt="" style="width:400px;">
</p>

As we can see based on these graphs and metrics, these are not great models for predicting a BYU student's favorite music genre.

# Random Forest

I created two random forest models to predict favorite music genre -- one with all of the variabels in the dataset [hereafter Model 3], and one with just the variables specified above [hereafter Model 4] -- using the sklearn `RandomForestClassifier()` package. In order to determine the best values for the hyperparameters `n_estimators` and `max_depth`, I used `GridSearchCV()`. For model 3, the best hyperparameter values were `n_estimators=300` (out of 100,200,300,400,500,600,700) and `max_depth=5` (out of None,5,10,15,20), and for model 2, the best values were `n_estimators=500` and `max_depth=5`.

Model 3 has an accuracy of 0.34545, while model 4 has an accuracy of 0.35909. The overall ROC AUC for model 3 is 0.66452, while the ROC AUC for model 4 is 0.64344. They are both better than either of the KNN models (models 1 and 2). Here are the one-vs-rest ROC curves for each music genre:

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/knn_model3_roc.png" alt="" style="width:400px;">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/knn_model4_roc.png" alt="" style="width:400px;">
</p>

As we can see based on these graphs and metrics, these models are slightly better than the KNN models, but they still are not great models for predicting a BYU student's favorite music genre.

## Feature Importance

The `RandomForestClassifer()` model allows us to look at feature importances. 

<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/feature_importances.png" alt="" style="width:600px;">

According to this, the top 7 most importance features for predicting a BYU student's favorite music genre are Sex, Salary, Movie, YearInternet, YHowTall, College, and Height, only 3 of which are part of the 7 variables I selected for my original research question. I'm curious to see how a model with just these features performs.

## Logistic Regression

Using sklearn's `LogisticRegression()` function, I created a model [hereafter Model 5] to predict Music using Sex, Salary, Movie, YearInternet, YHowTall, College, and Height, the features we selected above. In order to determine the best values for the hyperparameter `C`, I used `GridSearchCV()`, and the best value was `C=0.1`.

The model's accuracy is 0.29545, while its ROC AUC is 0.64391. Here are the one-vs-rest ROC curves for each music genre: 

<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/knn_model5_roc.png" alt="" style="width:400px;">

This is about the same as the KNN and RF models, so we still can't make very good predictions of a BYU student's favorite music genre.

## Ensemble Model

Let's try one last model to see if we can improve our predictions. This will be an ensemble of all the model types we've tried so far (KNN, Random Forest, and Logistic Regression). We will use all variables in this model.

The model's accuracy is 0.35909, while its ROC AUC is 0.6392.  Here is are the one-vs-rest ROC curves for each music genre:

<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/knn_model6_roc.png" alt="" style="width:400px;">

Unfortunately, we still can't make great predictions of a BYU student's favorite music genre. Since this seems to be a wash, let's look at what we *can* learn from the data with some unsupervised learning.

<a href="https://kbmoore02.github.io/Stat486-Final-Blog/2023/12/08/blog-unsupervised.html">Here</a> is the next part of my analysis.
