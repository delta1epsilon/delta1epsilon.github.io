---
title:  "Gotchas with Clustering"
date:   2019-06-07 23:04:23
categories: [clustering]
tags: [clustering, curse of dimensionality, high-dimensional data, clustering tendency, clustering evaluation, silhouette score, correlated features]
mathjax: true
---
Clustering is so extensively used technique in data science for so many kinds of applications. However, often times some important questions are being ignored in context of cluster analysis, which might lead to misleading results. In this post I would like to describe some of such questions.



## A black cat in a dark room - Are you sure it’s there?

It's tempting to start applying any clustering algorithm on your data set as soon as possible, quickly get some results and carry on your research. But are you sure there are any clusters, in your dataset, that you are interested in? Are you sure there's any non-random structure in your data? What if your data is uniformly distributed - would clustering still make any sense for you?

Clustering is meaningful only if there’s some non-random structure in a dataset, otherwise your results might be misleading. It’s worth investing some time in assessing clustering tendency of your dataset.

[Hopkins statistic](https://en.wikipedia.org/wiki/Hopkins_statistic) can be used for measuring clustering tendency - it tests whether a data is uniformly distributed.

It’s always a good idea to visualize your data. t-SNE might be used for visualizing high-dimensional data. The figure below shows t-SNE embeddings for one of my datasets used for clustering - we can conclude that there’re some structures in the datasets and can try applying clustering to identify and analyze them.

![](https://delta1epsilon.github.io/assets/gotchas_w_clustering/tsne_embeddings.png)





## What if the data is high-dimensional?

It’s kind of intuitive that gathering more features might provide new information. However, at the same time it can cause severe problems. What are the challenges that high-dimensional data presents?

Many of the dimensions are often irrelevant and clusters may exist only in some subspaces. Clusters can be spread out across irrelevant dimensions, which can confuse traditional clustering algorithms. So, if you consider all the dimensions, you may no be able to find meaningful clusters. A nice example of such case is described in the [paper](https://www.kdd.org/exploration_files/parsons.pdf).

![](https://delta1epsilon.github.io/assets/gotchas_w_clustering/high_dimension_clustering_1.png)


![](https://delta1epsilon.github.io/assets/gotchas_w_clustering/high_dimensional_clustering_2.png)

#TODO: briefly describe the example


Curse of dimensionality is named to be the biggest problem in machine learning after overfitting [^fn2]. As the number of dimensions increase, the data becomes more sparse and distance measure becomes increasingly meaningless. Points in very high dimensions tend to be equidistant from each other. [reference number for https://www.kdd.org/exploration_files/parsons.pdf, Figure 1] provides an example of how additional dimensions spread out the points - see the figure below (the dataset is a sample of 20 points from uniform distribution between 0 and 2 in each of three dimensions).


![](https://delta1epsilon.github.io/assets/gotchas_w_clustering/high_dimension_sparcity.png)


So, what are the alternative to traditional clustering algorithms for clustering high-dimensional datasets?

Subspace clustering can be used for clustering high-dimensional data. It extends traditional clustering algorithms by finding clusters in different subspaces of the given high-dimensional space, where a subspace is defined using a subset of attributes in the full space.

Also, dimensionality reduction approaches can be used to construct a much lower-dimensional space and search for clusters in such space.




## What about features scales?

Normally your dataset can contain features describing very diverse information in different scales and units. Using a distance measure, which doesn’t account for features spread, might lead to undesirable results, since features with higher magnitudes are gonna be more influential. For instance, Euclidean distance will be governed by the features that have high range of values.

Therefore, the features should be normalized so that each feature contributes approximately proportionately to the final distance.




## What if there’re correlated features?

Correlated features can affect some models in different ways. However, is it important in context of clustering? It depends on your distance measure, if it doesn’t encounter for features covariance, then yes - the distance measure can be affected.

The problem is that highly correlated features get higher weights in the distance measure than other features. The resulting distance is likely to be skewed in the direction of the concept represented by the highly correlated features.

As an example of extreme case of correlation, consider a case when there are two features representing the same information in different units, e.g. objects weight in kilos and in pounds. Using, for instance, Euclidean distance will turn out into weighted Euclidean distance with the concept of objects weight getting high weights because of the two perfectly correlated features.

There’re distance measures which are not influenced by highly correlated variables, for example [Mahalanobis distance](https://en.wikipedia.org/wiki/Mahalanobis_distance) takes into account the correlations of the dataset.

Check out a blog post [From the Euclidean distance over standardized variables to the Mahalanobis distance](https://milania.de/showcase/From_the_Euclidean_distance_over_standardized_variables_to_the_Mahalanobis_distance) - it contains a very nice visualization of the difference between Euclidean distance with/without standardization and Mahalanobis distance.





## What are the specifics of the picked clustering algorithm?

Different algorithms define and represent clusters in different ways. Some assign objects to belong to one of the clusters while other assign objects to belong to each cluster to a certain degree, clusters shapes can vary, some allow for noise and outliers to be assigned to separate category, some algorithms scale well while other don’t. Specifics of your clustering problem and the dataset should be aligned with a selected algorithm.

It's essential to understand details of algorithms of your choice. As an example, the most well-known and commonly used clustering algorithm - K-means has these specifics, which often times are not desirable:
* K-means can be applied only when the mean of a set of objects is defined 
* K-means is not suitable for discovering clusters of arbitrary shapes or clusters of very different sizes
* K-means is sensitive to noise and outliers
* K-means is not guaranteed to converge to the global optimum
* K-means require predefined number of clusters

[insert figure from https://scikit-learn.org/stable/auto_examples/cluster/plot_kmeans_assumptions.html]





## How to evaluate clustering results?

There is no recognized universal measure for evaluating clustering results. However, measures which consider how well the clusters are separated and how compact the clusters are can be used. (Silhouette coefficient)[https://en.wikipedia.org/wiki/Silhouette_(clustering)] is one of such measures - it measures how similar a sample is to other samples in the same cluster and how different it is to samples in other clusters. Silhouette coefficient is calculated for each sample in a dataset and its value is in between -1 and 1:
* Sample with value close to 1 is in compact and well separated cluster
* Sample with negative silhouette score is closer to samples from other clusters
* Values near 0 indicate overlapping clusters

As an example of silhouette analysis, consider the figure below, where the horizontal lines on the left side subplot are silhouette scores of all samples clustered on the right side figure:
[insert image https://scikit-learn.org/stable/_images/sphx_glr_plot_kmeans_silhouette_analysis_005.png]
We can see that a couple of clusters have silhouette scores smaller than average score (the red dashed line) and there’re some samples with negative scores. It points that the clustering is poorly performed and some changes should be made, e.g specified number of clusters is wrong. See the example in scikit-learn “Selecting the number of clusters with silhouette analysis on KMeans clustering”[https://scikit-learn.org/stable/auto_examples/cluster/plot_kmeans_silhouette_analysis.html#sphx-glr-auto-examples-cluster-plot-kmeans-silhouette-analysis-py].

Note that such clustering quality measures are just heuristics, it doesn’t guarantee meaningful clusters. Significance of the obtained clusters should be analyzed by domain experts.





## Summary







## References

[^fn1]: Subspace Clustering for High Dimensional Data: A Review [https://www.kdd.org/exploration_files/parsons.pdf](https://www.kdd.org/exploration_files/parsons.pdf)

[^fn2]: section "Intuition fails in high dimensions" in paper "A Few Useful Things to Know about Machine Learning" by Pedro Domingos [https://homes.cs.washington.edu/~pedrod/papers/cacm12.pdf](https://homes.cs.washington.edu/~pedrod/papers/cacm12.pdf)






