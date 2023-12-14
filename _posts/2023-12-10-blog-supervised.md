---
layout: post
title: "Supervised Learning Methods"
author: Kelsey Moore
description: Apply supervised learning to address the primary research question
image: /assets/images/sup_header.jpg
---

## Primary Research Question

My primary research goal is to see if I can predict a BYU student's favorite genre of music based on factors such as their major, their gender, if they played sports in high school, their favorite movie genre, what streaming service they use the most, their favorite social media app, and what part of the country they want to settle in.

## K-Nearest Neighbors

I created two knn models to predict favorite music genre -- one with all of the variables in the dataset [hereafter Model 1], and one with just the variables specified above [hereafter Model 2] -- using the sklearn `KNeighborsClassifier()` package. In order to determine the best values for the hyperparameters `n_neighbors` and `weights`, I used `GridSearchCV()`. For both models, the best hyperparameter values were `n_neighbors=2` (out of 1,2,3,4,5) and `weights='uniform'` (out of 'uniform' and 'distance').

Both models have the same accuracy of 0.27727

<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/knn_model1_roc.png" alt="" style="width:1000px;">
<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/knn_model2_roc.png" alt="" style="width:1000px;">
