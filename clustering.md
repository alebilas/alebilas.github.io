# Comparison of K-means++, PAM and hierarchical methods of clustering. Meat stores use case.
#### Aleksandra Biłas

## Introduction
The below clustering analysis was performed in order to create value-based groups of meat stores. 3 clustering methods are presented, using 2 types of input data: results of Principal Component Analysis and scaled original data. The results show that PCA outcomes are not always the best outpus to clustering due to very low silhouette scores returned, and simple scaling of original data may be enough for analyses.

#### Required libraries
```markdown
library(purrr)
library(stats)
library(factoextra)
library(caret)
library(cluster)
library(ClusterR)
library(LICORS)
library(tidyverse)
library(dendextend)
```

## Preliminary analysis
**PCA outcomes**

The first type of input tested were results obtained in PCA analysis and limited to only those Principal Components that showed significancy in at least one rotated loading during an interpretation phase. Thus, we have got the following attributes to be taken into account:
**PC1**: Items used on a daily basis. Products for breakfast, lunch.

**PC4**: Juices, sweets.

**PC2**: Beef meat.

**PC20**: Other meat, usually used for soups.

**PC13, PC3, PC9, PC23**: Meal sides, conserves.

**PC16, PC11, PC19**: Ready-made food.

**PC8**: Oils.

**PC20, PC18**: Seasonings.

**PC6**: Raw salads.

**PC22**: Smoked bones.


**Original data**
The initial dataset numbers 208 rows (stores) and 52 attributes - ID and 51 attributes for revenue referring to 51 product categories. The first step is to check whether all columns may be used in terms of variance and null rate. 

```markdown
summary(df)
```
4 columns have been removed due high null rate (greater than 75%): Alkohole&wina (alkohol&wine), Mrozonki&lody (frozen foods, ice creams), Pieczywo (breadstuff), Pozostale Przyprawy (other seasonings). 

As the dataset contains pieces of information about revenues only, the units for all columns are the same, so there is no need to scale data. Due to small sample size, outliers are not removed.

## Analysis
### K-means++
K-means++ is proven by many authors to be more effective than standard K-means due to more stable centroids. The difference between K-means and K-means++ is that in the former initial centers are chosen randomly (as in the latter) but then subsequent centers are chosen from among remaining data points with probability corresponding to distance between a data point and the previous center. In other words, K-means++ centroids are less random than in case of the standard K-Means algorithm.

_Searching for an optimal number of clusters_

Clustering methods usually require to set a parameter for number of clusters to produce. To reduce number of attempts one can use a function that depicts silhouette measures for variuos numbers of clusters.

```markdown
# Optimal number of clusters - silhouette score (PCA data)
opt_km_pca <- Optimal_Clusters_KMeans(df_pca_l, max_clusters=10, plot_clusters=TRUE, criterion="silhouette")
```
![Optimal number of clusters for K-means on PCA data](https://github.com/alebilas/images/blob/main/pca_kmeans_optimal_clust_number_sil.png)

```markdown
# Optimal number of clusters - silhouette score (original data)
opt_km<-Optimal_Clusters_KMeans(df.s, max_clusters=10, plot_clusters=TRUE, criterion="silhouette")
```
![Optimal number of clusters for K-means on original data](https://github.com/alebilas/images/blob/main/org_kmeans_optimal_clust_number_sil.png)

In the pictures above one can see that 2 clusters give the highest silhouette score which is quite expected because usually the highest number of clusters the lowest silhouette score or other quality measures. Becasue of that I decided to choose the number of clusters based on the lowest delta resulted from adding additional cluster which 3 for PCA data. For original data I keep 2 cluster as suggested by the plot because the drop between 2th and 3rd cluster if too laarge.

_Running K-means++ algorithm_
```markdown
################################## PCA data ##################################

# Train K-Means++
kmpp_pca <- kmeanspp(df_pca_l, k = 3, start = "random")

# Show cluster plot: dim1, dim2
fviz_cluster(list(data=df_pca_l, cluster=kmpp_pca$cluster), 
             ellipse.type="norm", geom="point", stand=FALSE, palette="jco", ggtheme=theme_classic())

# Show silhouette plot
sil_pca_kmpp <- silhouette(kmpp_pca$cluster, dist(df_pca_l))
fviz_silhouette(sil_pca_kmpp)

##############################################################################
```
![Clusters visualization for K-means++ on PCA data](https://github.com/alebilas/images/blob/main/fviz_cluster_kmpp_pca.png)

![Silhouette scores visualization for K-means++ on PCA data](https://github.com/alebilas/images/blob/main/kmpp_pca_sil_vis.png)

| cluster | size | ave.sil.width |
| ------- | ---- | ------------- |
|       1 | 121  |        0.26   |
|       2 |  64  |       -0.10   |
|       3 |  23  |       -0.14   |


```markdown
################################ Original data ###############################

# Train K-means++
kmpp <- kmeanspp(df, k = 2, start = "random")

# Show cluster plot: dim1, dim2 (Origional data)
fviz_cluster(list(data=df, cluster=kmpp$cluster), 
             ellipse.type="norm", geom="point", stand=FALSE, palette="jco", ggtheme=theme_classic())

# Show silhouette plot (Original data)
 sil<-silhouette(kmpp$cluster, dist(df))
fviz_silhouette(sil)

##############################################################################
```
![Clusters visualization for K-means++ on original data](https://github.com/alebilas/images/blob/main/fviz_cluster_kmpp_org.png)

![Silhouette scores visualization for K-means++ on original data](https://github.com/alebilas/images/blob/main/kmpp_org_sil_viz.png)


| cluster | size | ave.sil.width |
| ------- | ---- | ------------- |
|       1 |  148 |       0.53   |
|       2 |  60  |       0.21   |

**Conclusion:** K-means++ powered by original data presents much better results in terms of silhouette score and rarely overlapping clusters.



### Partitioning Around Medoids
PAM algorithm is alike K-means with a difference of looking for initial clusters among real data points, not like in K-means - selecting them randomly.

_Searching for an optimal number of clusters_
```markdown
# Optimal number of clusters - silhouette score (PCA data)
opt_md_pca <- Optimal_Clusters_Medoids(df_pca_l, max_clusters=10, 'euclidean', plot_clusters=TRUE, criterion="silhouette")
```
![Optimal number of clusters for PAM, PCA](https://github.com/alebilas/images/blob/main/opt_clust_num_pam_pca.png)

```markdown
# Optimal number of clusters - silhouette score (Original data)
opt_md<-Optimal_Clusters_Medoids(df, max_clusters=10, 'euclidean', plot_clusters=TRUE, criterion="silhouette")
```
![Optimal number of clusters for PAM, original data](https://github.com/alebilas/images/blob/main/opt_clust_num_pam_org.png)

In PAM using PCA data I will follow the same logic as in K-means++ and choose k = 3 for PCA-based experiment. However the plot for PAM based on original data shows a big drop from 2 to 3 clusters, thus 2 clusters will be generated.

_Running PAM algorithm_

```markdown
################################## PCA data ##################################

# Train PAM and show summary
pam_pca <- pam(df_pca_l, 3)
summary(pam_pca)

# Show cluster plot: dim1, dim2
fviz_cluster(pam_pca, geom = "point", ellipse.type = "convex")

# Show silhouette plot
sil_pca_pam <- silhouette(pam_pca$cluster, dist(df_pca_l))
fviz_silhouette(sil_pca_pam)

##############################################################################
```

Average silhouette width per cluster:

0.03363569 0.02823446 0.04791684

![Clusters visualization for PAM on PCA data](https://github.com/alebilas/images/blob/main/fviz_cluster_pam_pca.png)

![Silhouette scores visualization for PAM on PCA data](https://github.com/alebilas/images/blob/main/sil_plot_pam_pca.png)

```markdown
################################ Original data ###############################

# Train PAM and show summary
pam <- pam(df, 2)
summary(pam)

# Show cluster plot: dim1, dim2
fviz_cluster(pam, geom = "point", ellipse.type = "convex")

# Show silhouette plot
sil_pam <- silhouette(pam$cluster, dist(df))
fviz_silhouette(sil_pam)

##############################################################################
```

Average silhouette width per cluster:

0.5656762 0.1312273

![Clusters visualization for PAM on originl data](https://github.com/alebilas/images/blob/main/fviz_cluster_pam_org.png)

![Silhouette scores visualization for PAM on original data](https://github.com/alebilas/images/blob/main/sil_plot_pam_org.png)

**Conclusion:** Again, original data gives much better results than PCA input. Moreover, in the winning approach, compared to K-means++ resulting clusters are less overlapped which is also reflected in higher silhouette score.

### Hierarchical algorithms
The last part of the analysis was a test for hierarchical methods quality. First, one can check which of various hierarchical methods produces the higest agglomerative coefficient.


```markdown
################################## PCA data ##################################

ac_pca <- function(x) {
  agnes(df_pca_l, method = x)$ac
}
map_dbl(m, ac_pca)

##############################################################################
```

|  average  |   single  |  complete |   ward    |
| --------- | --------- | --------- | --------- |
| 0.7015815 | 0.6623972 | 0.7900627 | **0.8008557** | 


```markdown

################################ Original data ###############################

ac <- function(x) {
  agnes(df, method = x)$ac
}
map_dbl(m, ac)

##############################################################################

```

|  average  |   single  |  complete |   ward    | 
| 0.8807898 | 0.8791851 | 0.9114954 | **0.9652859** |

In both cases AC suggests using Ward method.

_Running Ward algorithm_

```markdown
################################## PCA data ##################################

# Train hierarchical clustering - Ward
hc_pca <- hcut(df_pca_l, k = 3, hc_method = "ward.D2")

# Dendrogram
fviz_dend(hc_pca, rect = TRUE)

# Silhouette plot
fviz_silhouette(hc_pca)

##############################################################################
```

| cluster | size | ave.sil.width |
| ------- | ---- | ------------- |
|       1 |  39  |       -0.14   |
|       2 | 161  |        0.30   |
|       3 |   8  |       -0.14   |

![Dendrogram for HC on PCA data](https://github.com/alebilas/images/blob/main/hc_pca_dendr.png)

![Silhouette scores visualization for HC on PCA data](https://github.com/alebilas/images/blob/main/hc_pca_sil.png)


```markdown
################################ Original data ###############################

# Train hierarchical clustering - Ward
hc <- hcut(df, k = 3, hc_method = "ward.D2")

# Dendrogram
fviz_dend(hc, rect = TRUE)

# Silhouette plot
fviz_silhouette(hc)

##############################################################################
```

| cluster | size | ave.sil.width |
| ------- | ---- | ------------- |
|       1 | 111  |        0.14   |
|       2 |  71  |        0.50   |
|       3 |  26  |        0.20   |

![Dendrogram for HC on original data](https://github.com/alebilas/images/blob/main/hc_org_dendr.png)

![Silhouette scores visualization for HC on original data](https://github.com/alebilas/images/blob/main/hc_org_sil.png)

**Conclusion:** For the third time leveraging original data gives better results, so the choise is simple as to selection of input data. From the algorithm point of view, PAM managed best what can be seen the highest positive silhouette scores. Clusters generated by PAM are added to the main dataset to be analysed further in terms of variables means.

```markdown
df_pam <- cbind(df_with_id, pam$clustering)
names(df_pam)[names(df_pam) == 'pam$clustering'] = 'cluster'

# Generate mean values of each variable grouped by a cluster
aggr = as.data.frame(aggregate(df_pam[,2:48], by = list(df_pam$cluster), FUN = mean))
View(aggr)
```

**Final conclusion:** The above analysis aimed at testing 3 different methods for 2 input sets of real life data. In all situations original, not scaled data turned out to give significantly better results than Principal Components created leveraging the same data frame. As PCA is very popular method, one must always consider that it is not one-size-fits-all approach. Among 3 implemented algorithms, Partitioning Around Medoids has given the most trustworthy results, hence the further analysis of basic statistics was prepared on that basis. Mean values of majority variabled indicated that the 1st cluster represents stores with revenues below population average, whereas the 2nd cluster collects stored performing better than the average units.




