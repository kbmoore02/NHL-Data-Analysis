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
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/cluster1.png" alt="" style="width:200px;">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/cluster2.png" alt="" style="width:200px;">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/cluster3.png" alt="" style="width:200px;">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/cluster4.png" alt="" style="width:200px;">
</p>

Based on these, I decided that `n_clusters=4` is the best value. Using sklearn's `GaussianMixture()` algorithm, I found 4 clusters in the data. The following are representations of how the data fits into these clusters:

<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/meta-chart.png" alt="" style="width:300px;">
<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/all_clusters1.png" alt="" style="width:1000px;">

Looking at this graph, the only really noticeable trends are in height and shoe size, which I presume is due to typical differences between men and women. We know from our random forest model that sex is the most important factor in predicting music taste, so it makes sense that it would be a defining characteristic in separating the clusters.

<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/all_clusters2.png" alt="" style="width:1000px;">
<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/all_clusters3.png" alt="" style="width:800px;">

Looking at these graphs, we see a couple of interesting patterns:
- Most people in a relationship are in cluster 3
- Most people who use dating apps are in cluster 1
- The people who spend the most on food each week are in clusters 3 and 4
- Cluster 3 has the highest median height, shoe size, and salary
- Most people who played sports in high school are in cluster 2
- Most people in the college of education are in cluster 1
- Most people in the nursing program and in the college of humanities are in cluster 4
- All people with an Apple phone are in cluster 2
- Most men are in clusters 1 and 4 and most women are in clusters 2 and 3
- Most people who want to live in the western US are in cluster 1
- Most people who prefer YouTube are in cluster 3
- Most people who prefer foreign films are in cluster 4
- Most people who prefer Netflix are in cluster 1

# Dimension Reduction

Now I'm going to perform some dimension reduction so that we can get a look at these clusters all together. I did both sklearn's `PCA()` and `TSNE()` with 2 components, so it would be easy to visualize. Here are the results:

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/pca.png" alt="" style="width:400px;">
  <img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/tsne.png" alt="" style="width:400px;">
</p>

The clusters are much more defined for PCA than for tSNE. When looking at tSNE, the most defined clusters are 3 and 4 which makes sense, because those are the largest clusters. Looking at the PCA clusters, 1 and 4 overlap a lot, while 2 and 3 are a bit more separate. This pattern is reflective of how most men are in clusters 1 and 4, and most women are in clusters 2 and 3, as we saw above in our cluster analysis. Let's look at these clusters based on gender:

<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/pca_sex1.png" alt="" style="width:400px;">

The separation between the two is very clear. So it seems, just as we discovered with our supervised learning, that gender is the most important factor in this dataset.  
