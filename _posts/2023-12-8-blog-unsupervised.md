---
layout: post
title: "Unsupervised Learning Methods"
author: Kelsey Moore
description: Apply unsupervised learning to student data
image: /assets/images/unsup_header.png
---

# Introduction

Now that we've determined that we can't predict music taste with much success, let's see what we *can* learn from this data. I'm going to apply some unsupervised learning models and see what information we can extract from this data.

# Cluster Analysis

I started by computing a few different metrics to see how many clusters I should look for.

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/cluster1.png" alt="" style="width:230px;">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/cluster2.png" alt="" style="width:230px;">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/cluster3.png" alt="" style="width:230px;">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/cluster4.png" alt="" style="width:230px;">
</p>

Based on these, I decided that `n_clusters=4` is the best value. Using sklearn's `GaussianMixture()` algorithm, I found 4 clusters in the data. The following are representations of how the data fits into these clusters:

<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/meta-chart.png" alt="" style="width:300px;">
<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/all_clusters1.png" alt="" style="width:1000px;">

Looking at this graph, the only really noticeable trends are in height and shoe size, which I presume is due to typical differences between men and women. We know from our random forest model that sex is the most important factor in predicting music taste, so it makes sense that it would be a defining characteristic in separating the clusters.

<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/all_clusters2.png" alt="" style="width:1000px;">

