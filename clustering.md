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
