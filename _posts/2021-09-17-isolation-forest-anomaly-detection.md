---
layout: post
categories: "en"
title: "Isolation Forest"
date: 2021-09-17
tags: ml R
desc: "Using Isolation Forest for Anomaly Detection with h2o.ai library"
comment_id: 10

---

Anomaly or outlier detection is the search of atypical observations that don't seem to fit with a pattern found in the majority of data. To start an anomaly detection project is tricky, because usually the source data is not labelled as 'anomaly'/'non-anomaly', so unsupervised methods are the go-to method to identify outliers.

There are at least two broad approaches to search for  these rare observations, one is based on defining a 'normal' pattern first, and then ruling out the specific cases that don't conform to that pattern.

The other approach, the one I'm interesting in using, is not concerned with the pattern of the majority of the data, but with finding ways to _isolate_ observations. The idea of an isolation tree is to build a tree following the same splits as a common classification tree. When the tree is fully grown, reaching one observation in each leaf, we can then measure how long it took for the tree to reach that one observation.

Looking at the following chart, we can see that adjusting a tree with all leaves of size = 1 will provide different number of splits for different observations. The pink path is the shortest among all leaves, and thus can be identified as an anomaly. The other two blue leaves have 4 splits, and as the trees continues, we can see that for the rest of the observations the path length is at least 5.

<br>
<img src="/images/isolation_tree.png" class="center" height=480px>
<br>


The shorter the path between the origin and a leaf, the more rare that observation is. A measure for the path distance is the _number of splits needed_ to isolate a single observation.

---


Finding anomalies is of great interest in OLAP data, so I adjusted my Isolation Tree using an [open kaggle data set](https://www.kaggle.com/arjunbhasin2013/ccdata) of credit card transactions. The data contains usage behavior of about 9000 active credit card holders during the last 6 months. After running the isolation tree, most detected outliers include high amounts of payments and purchases.

The library used to run my Isolation Tree is my favorite [h20.ai 💛](https://www.h2o.ai/) , but isolation forests are also [available at scikitlearn](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.IsolationForest.html). I like h20.ai because it's an open source, distributed, fast, and flexible machine learning platform that runs on **both R and Python**, which is a great relief for statisticians still undecided on which tool is best.

For a quick, 20ish line script, R and h20.ai do an amazing job in detecting the outliers.

<pre>
library(h2o)

options(scipen=999)

# Read file from Kaggle Open Data
cc <- read.csv('CC GENERAL.csv', dec = '.')

# Forget NAs
cc <- cc[is.na(cc$CREDIT_LIMIT)==F,]
cc <- cc[is.na(cc$MINIMUM_PAYMENTS)==F,]
</pre>

When working with h2o.ai, first we have to initiate h2o and once it's loaded (takes 10-20 seconds for me), we can use the package.

<pre>
# Use h2o object
h2o.init()
df  <- as.h2o(cc)

# Build Isolation Tree
it  <- h2o.isolationForest(training_frame = df,
                          sample_rate = 0.1,
                          max_depth = 20,
                          ntrees = 1000)

# Keep prediction and mean length and add to original df
pred  <- h2o.predict(it, df)
cc_df <- cbind(cc,as.data.frame(pred))

# Distribution of mean length and predicted values:

hist(cc_df$predict, col='grey75', border='grey65', main = 'iTree predicted values', xlab = 'Predicted')
abline(v = 0.85, col='orange', lwd=3, lty=3)
</pre>

<br>
<img src="/images/itree_histogram.png" class="center" height=380px>
<br>

After watching the histogram, I chose a cut of 0.85 that separates <4% of the observations. This gives a small set of anomalies to investigate and label, simplifying the task of detecting anomalies by providing a small set of observations to look into.

Partial plots of some X variables and the predicted values show that the prediction tree usually isolates observations with hight minimum payments and usage rates, which is the standard in the industry.



<br>
<img src="/images/itree_scatter1.png" class="center" height=380px>
<br>

<br>
<img src="/images/itree_scatter3.png" class="center" height=380px>
<br>


My [repo of h2o.ai stuff I like is on github](https://github.com/camilaburne/h2o), with the isolation tree case included. Of course, as any tree, there's a lot of parameters that can be calibrated, and I think that the good practices for random forest are also applicable here. I just tried to run a quick example to start getting to know it. What I like about the algorithm is the novelty in its application, what was previously mostly used for supervised classification is useful in an unsupervised problem of detecting outliers.




<br>

### Resources on isolation forest:
- [Original paper by: Liu, Fei Tony, Ting, Kai Ming, and Zhou, Zhi-Hua, “Isolation Forest”](https://cs.nju.edu.cn/zhouzh/zhouzh.files/publication/icdm08b.pdf)

- [h2o.ai documentation on Isolation Forest](https://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/if.html)

- [scikit documentation on Isolation Forest](https://scikit-learn.org/stable/auto_examples/ensemble/plot_isolation_forest.html#sphx-glr-auto-examples-ensemble-plot-isolation-forest-py)

<br>
