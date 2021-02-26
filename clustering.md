## Comparison of K-means++, PAM and hierarchical methods of clustering. Meat stores use case.
#### Aleksandra Biłas

### Introduction
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

### Preliminary analysis
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

### Analysis
#### K-means++
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

_K-means++ algorithm_
```markdown
### Run K-means++ from LICORS library ###
# PCA data
kmpp_pca <- kmeanspp(df_pca_l, k = 3, start = "random")

# Original data
kmpp <- kmeanspp(df, k = 2, start = "random")

_Clusters and silhouette plots_
# Cluster plot: dim1, dim2 (PCA data)
fviz_cluster(list(data=df_pca_l, cluster=kmpp_pca$cluster), 
             ellipse.type="norm", geom="point", stand=FALSE, palette="jco", ggtheme=theme_classic())

# Silhouette plot (PCA data)
sil_pca_kmpp <- silhouette(kmpp_pca$cluster, dist(df_pca_l))
fviz_silhouette(sil_pca_kmpp)

# Cluster plot: dim1, dim2 (Origional data)
fviz_cluster(list(data=df, cluster=kmpp$cluster), 
             ellipse.type="norm", geom="point", stand=FALSE, palette="jco", ggtheme=theme_classic())

# Silhouette plot (Original data)
 sil<-silhouette(kmpp$cluster, dist(df))
fviz_silhouette(sil)

```
Plots for algorithms based on PCA data
![Clusters visualization for K-means++ on PCA data](https://github.com/alebilas/images/blob/main/fviz_cluster_kmpp_pca.png)

![Silhouette scores visualization for K-means++ on PCA data](https://github.com/alebilas/images/blob/main/kmpp_pca_sil_vis.png)

Plots for algorithms based on original data
![Clusters visualization for K-means++ on original data](https://github.com/alebilas/images/blob/main/fviz_cluster_kmpp_org.png)

![Silhouette scores visualization for K-means++ on original data](https://github.com/alebilas/images/blob/main/kmpp_org_sil_viz.png)


             PCA data          
| cluster | size | ave.sil.width |
| ------- | ---- | ------------- |
|       1 | 121  |        0.26   |
|       2 |  64  |       -0.10   |
|       3 |  23  |       -0.14   |

          Original data          
| cluster | size | ave.sil.width |
| ------- | ---- | ------------- |
|       1 |  148 |       0.53   |
|       2 |  60  |       0.21   |

K-means++ powered by original data presents much better results in terms of silhouette score.

#### Partitioning Around Medoids
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

In PAM using PCA data I will follow the sme logic as in K-means++ and choose k = 3. However in PAM based on original data the drop from 2 to 3 clusters is too high to repeat the approach, thus 2 clusters will be generated.

_PAM algorithm_
```markdown
# Run PAM (PCA data)
pam_pca <- pam(df_pca_l, 3)
summary(pam_pca)

# Run PAM (Original data)
pam <- pam(df, 2)
summary(pam)

_Clusters and silhouette plots_
# Cluster plot: dim1, dim2
fviz_cluster(pam_pca, geom = "point", ellipse.type = "convex")

# Silhouette plot
sil_pca_pam <- silhouette(pam_pca$cluster, dist(df_pca_l))
fviz_silhouette(sil_pca_pam)
```
Silhouette score (PCA data)
Average silhouette width per cluster:
0.03363569 0.02823446 0.04791684

Silhouette score (Original data)
Average silhouette width per cluster:
0.5656762 0.1312273

Cluster and silhouette plots (PCA data)
![Clusters visualization for PAM on PCA data](https://github.com/alebilas/images/blob/main/fviz_cluster_pam_pca.png)

![Silhouette scores visualization for PAM on PCA data](https://github.com/alebilas/images/blob/main/sil_plot_pam_pca.png)

Cluster and silhouette plots (Original data)
![Clusters visualization for PAM on originl data](https://github.com/alebilas/images/blob/main/fviz_cluster_pam_org.png)

![Silhouette scores visualization for PAM on original data](https://github.com/alebilas/images/blob/main/sil_plot_pam_org.png)

Again, original data gives much better results than PCA input.

#### Hierarchical algorithms
The 3rd part of the analysis was a test for hierarchical methods quality. First, one can check which of various hierarchical methods produces the higest agglomerative coefficient.

```markdown
ac_pca <- function(x) {
  agnes(df_pca_l, method = x)$ac
}
map_dbl(m, ac_pca)

ac <- function(x) {
  agnes(df, method = x)$ac
}
map_dbl(m, ac)
```

PCA data
|  average  |   single  |  complete |   ward    |
| --------- | --------- | --------- | --------- |
| 0.7015815 | 0.6623972 | 0.7900627 | **0.8008557** | 

Original data
|  average  |   single  |  complete |   ward    | 
| 0.8807898 | 0.8791851 | 0.9114954 | **0.9652859** |

AC suggests using Ward method, ergo:
```markdown
hc_pca <- agnes(df_pca_l, method = "ward")
pltree(hc_pca, cex = 0.6, hang = -1, main = "dendrogram - agnes")
rect.hclust(hc_pca, k = 3, border = 2:5)
```

_Ward method application_
```markdown
# Run hierarchical clustering - Ward (PCA data)
hc_pc <- hcut(df_pca_l, k = 3, hc_method = "ward.D2")

# Run hierarchical clustering - Ward (Original data)
hc <- hcut(df, k = 3, hc_method = "ward.D2")

_Dendrogram and silhouette plots_
# Dendrogram (PCA data)
fviz_dend(hc_pca, rect = TRUE)

# Dendrogram (Originaal data)
fviz_dend(hc, rect = TRUE)

# Silhouette plot (PCA data)
fviz_silhouette(hc_pca)

# Silhouette plot (Original data)
fviz_silhouette(hc)
```

PCA data
| cluster | size | ave.sil.width |
| ------- | ---- | ------------- |
|       1 |  39  |       -0.14   |
|       2 | 161  |        0.30   |
|       3 |   8  |       -0.14   |

Original data
| cluster | size | ave.sil.width |
| ------- | ---- | ------------- |
|       1 | 111  |        0.14   |
|       2 |  71  |        0.50   |
|       3 |  26  |        0.20   |

Dendrogram and silhouette plot (PCA data)
![Dendrogram for HC on PCA data](https://github.com/alebilas/images/blob/main/hc_pca_dendr.png)

![Silhouette scores visualization for HC on PCA data](https://github.com/alebilas/images/blob/main/hc_pca_sil.png)


Dendrogram and silhouette plot (Original data)
![Dendrogram for HC on original data](https://github.com/alebilas/images/blob/main/hc_org_dendr.png)

![Silhouette scores visualization for HC on original data](https://github.com/alebilas/images/blob/main/hc_org_sil.png)

For the 3rd time leveraging original data gives better results, so the choise is simple as to selection of input data. From the algorithm point of view, PAM managed best what can be seen the highest positive silhouette scores.





