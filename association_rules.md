# What do meat-eaters buy together? Association rules on meat stores purchases patterns.
#### Aleksandra Biłas

## Introduction
Association rules are a set of methods which may easily describe patterns of customers purchases. They are widely used by FMCG companies as a recommendation engine input on websites, showing products that people similar to a current web-surfer also boughts. However for that companies usually need customers identifiers which is not always possible when a firm does not have a loyalty programme. In that case association rules might be leveraged as inbound promotions in the shape of pop-ups (recommendations) in Point of Sales.

### Required packages
```markdown
library(arules)
library(arulesViz)
library(arulesCBA)
library(dplyr)
library(tidyverse)
```

## Data preparation
An initial dataset contains over 5.3M items sold in more than 200 meat stores. Each item is linked to 2 categories groups which are high and medium level of granularity. A preliminary run of asscoation rules using SKU and the medium-level groups did not return satisfing results, so new categories for 1600 items has been manually created, giving over 70 new groups of products. Two extra steps consisted in removing shopping bags which were also a part of transactions, as well as transactions were only one product was bought.
