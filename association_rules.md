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

_Final dataset before transforming to AR-required format_
```markdown
head(trans_to_cvs)

write.csv(trans_to_csv, ".../transactions.csv")
```

|  uid_paragon |  nazwa_new |
| ------------ | ---------- |
| 100013591    | wedliny_wedzonki_kurczak |
| 8859881      | wedliny_karkowka |
| 13586882     | wedliny_metki |
| 100013592    | dodatki_maczne |
| 11407991     | wedliny_wedzonki_kurczak |
| 13586884     | wedliny_wedzonki_kurczak |

_Final dataset after transforming to AR-required format_
```markdown
# uid_paragon - unique identifier of a receipt; nazwa_new - newly created category
trans <- read.transactions(".../transactions.csv", format="single", sep=",", cols=c("uid_paragon","nazwa_new"), header=TRUE)

# Limiting transactions to those where there are at least 2 items
trans2 <- trans[size(trans) > 1]
inspect(head(trans2))

> inspect(head(trans2))
    items                        transactionID
[1] {garmazerka_pierogi,                      
     wedliny_kielbasa_sucha,                  
     wedliny_parowki,                         
     wedliny_salami,                          
     wedliny_schab,                           
     wedliny_szynka}               1000135910 
[2] {wedliny_metki,                           
     wedliny_podrobowe_inne}       10001359100
[3] {wedliny_kielbasa_sucha,                  
     wedliny_poledwica}            1000135911 
[4] {wedliny_metki,                           
     wedliny_szynka}               1000135914 
[5] {mieso_wieprzowina,                       
     wedliny_galarety,                        
     wedliny_podrobowe_salceson,              
     wedliny_salami,                          
     wedliny_szynka}               1000135917 
[6] {nabial_ser,                              
     wedliny_kielbasa_sucha}       1000135918     
```

```markdown
# Descriptive statistics
round(itemFrequency(trans2), 3)
itemFrequency(trans2, type="absolute")

> round(itemFrequency(trans2), 3)
            dodatki_dodatki                dodatki_inne               dodatki_kasze                dodatki_lody 
                      0.033                       0.003                       0.001                       0.000 
             dodatki_maczne            dodatki_mrozonki             dodatki_surowki               dodatki_syrop 
                      0.011                       0.002                       0.039                       0.004 
                 garmazerka            garmazerka_flaki             garmazerka_inne           garmazerka_maczne 
                      0.031                       0.011                       0.010                       0.018 
         garmazerka_pierogi             mieso_cielecina          mieso_garmazeryjne                 mieso_grill 
                      0.076                       0.004                       0.008                       0.003 
                mieso_indyk                mieso_kaczka               mieso_kurczak             mieso_pozostale 
                      0.067                       0.015                       0.224                       0.001 
          mieso_wieprzowina              mieso_wolowina                 nabial_inne                  nabial_ser 
                      0.228                       0.071                       0.009                       0.156 
           nabial_ser_bialy             nabial_ser_inne        nabial_ser_plesniowy               nabial_smalec 
                      0.001                       0.000                       0.003                       0.004 
                      oleje         pozostale_akcesoria           pozostale_alkohol              pozostale_inne 
                      0.000                       0.000                       0.000                       0.000 
             pozostale_miod            pozostale_nabial     pozostale_napoje_cieple      pozostale_napoje_zimne 
                      0.000                       0.000                       0.000                       0.005 
         pozostale_pieczywo         pozostale_przekaski         pozostale_przyprawy          pozostale_slodycze 
                      0.003                       0.000                       0.000                       0.001 
              pozostale_sos     pozostale_warzywa_owoce              przyprawy_inne             przyprawy_kamis 
                      0.035                       0.000                       0.000                       0.020 
      przyprawy_kamis_mieso         przyprawy_kamis_sos          przyprawy_marynaty          przyprawy_podravka 
                      0.008                       0.007                       0.001                       0.016 
   przyprawy_podravka_mieso             wedliny_baleron              wedliny_boczek            wedliny_galarety 
                      0.001                       0.023                       0.059                       0.036 
               wedliny_inne            wedliny_kabanosy wedliny_kanapkowe_driobiowe   wedliny_kanapkowe_parzone 
                      0.012                       0.047                       0.078                       0.112 
    wedliny_kanapkowe_suche            wedliny_karkowka            wedliny_kielbasa    wedliny_kielbasa_parzona 
                      0.136                       0.017                       0.000                       0.081 
     wedliny_kielbasa_sucha       wedliny_kości_wedzone               wedliny_metki             wedliny_parowki 
                      0.177                       0.004                       0.087                       0.160 
     wedliny_podrobowe_inne  wedliny_podrobowe_kaszanka   wedliny_podrobowe_pasztet  wedliny_podrobowe_salceson 
                      0.070                       0.021                       0.088                       0.056 
          wedliny_poledwica              wedliny_salami               wedliny_schab              wedliny_surowe 
                      0.048                       0.052                       0.192                       0.008 
             wedliny_szynka      wedliny_wedzonki_indyk    wedliny_wedzonki_kurczak 
                      0.377                       0.110                       0.111 


> itemFrequency(trans2, type="absolute")
            dodatki_dodatki                dodatki_inne               dodatki_kasze                dodatki_lody 
                      33372                        2848                        1026                          10 
             dodatki_maczne            dodatki_mrozonki             dodatki_surowki               dodatki_syrop 
                      10748                        1670                       39079                        4086 
                 garmazerka            garmazerka_flaki             garmazerka_inne           garmazerka_maczne 
                      30583                       11146                        9645                       18455 
         garmazerka_pierogi             mieso_cielecina          mieso_garmazeryjne                 mieso_grill 
                      75767                        4442                        7572                        2859 
                mieso_indyk                mieso_kaczka               mieso_kurczak             mieso_pozostale 
                      66536                       15279                      224032                         651 
          mieso_wieprzowina              mieso_wolowina                 nabial_inne                  nabial_ser 
                     228273                       70868                        8542                      155624 
           nabial_ser_bialy             nabial_ser_inne        nabial_ser_plesniowy               nabial_smalec 
                        509                         301                        3179                        4242 
                      oleje         pozostale_akcesoria           pozostale_alkohol              pozostale_inne 
                        325                         368                          13                         275 
             pozostale_miod            pozostale_nabial     pozostale_napoje_cieple      pozostale_napoje_zimne 
                         15                          76                         104                        4564 
         pozostale_pieczywo         pozostale_przekaski         pozostale_przyprawy          pozostale_slodycze 
                       3132                          68                          26                         595 
              pozostale_sos     pozostale_warzywa_owoce              przyprawy_inne             przyprawy_kamis 
                      34984                         115                         384                       20181 
      przyprawy_kamis_mieso         przyprawy_kamis_sos          przyprawy_marynaty          przyprawy_podravka 
                       7681                        6557                         728                       16264 
   przyprawy_podravka_mieso             wedliny_baleron              wedliny_boczek            wedliny_galarety 
                       1060                       23236                       58702                       36149 
               wedliny_inne            wedliny_kabanosy wedliny_kanapkowe_driobiowe   wedliny_kanapkowe_parzone 
                      12273                       46794                       77801                      111690 
    wedliny_kanapkowe_suche            wedliny_karkowka            wedliny_kielbasa    wedliny_kielbasa_parzona 
                     136364                       17111                           3                       80705 
     wedliny_kielbasa_sucha       wedliny_kości_wedzone               wedliny_metki             wedliny_parowki 
                     177242                        4369                       87085                      159884 
     wedliny_podrobowe_inne  wedliny_podrobowe_kaszanka   wedliny_podrobowe_pasztet  wedliny_podrobowe_salceson 
                      69662                       21320                       88081                       56301 
          wedliny_poledwica              wedliny_salami               wedliny_schab              wedliny_surowe 
                      47975                       51703                      192194                        8179 
             wedliny_szynka      wedliny_wedzonki_indyk    wedliny_wedzonki_kurczak 
                     376329                      110339                      110690
```

## Creating association rules
Receipts being under analysis come from all existing stores. Thus, setting standard parameters of _support_ = 0.1 and _confidence_ = 0.8 may lead to 0 rules, what actually happened. That's why, with trials and error, the final setting aare _support_ = 0.04 and _confidence_ = 0.3. This approach gave 11 rules. (Un)fortunately all right hand side elements contain only one components, and there are no pairs of item which could suggest any associated product, but I accept this approach for experimental goals.

```markdown
# generte rules
rules.trans<-apriori(trans2, parameter=list(supp=0.04, conf=0.3))

# Sort and show rules
rules.by.conf<-sort(rules.trans, by="confidence", decreasing=TRUE)
inspect(rules.by.conf)

> inspect(rules.by.conf)
     lhs                           rhs                 support    confidence coverage  lift      count 
[1]  {wedliny_schab}            => {wedliny_szynka}    0.08362447 0.4347794  0.1923377 1.1544544  83562
[2]  {wedliny_kanapkowe_suche}  => {wedliny_szynka}    0.05811641 0.4258675  0.1364659 1.1307909  58073
[3]  {nabial_ser}               => {wedliny_szynka}    0.06587221 0.4229618  0.1557403 1.1230753  65823
[4]  {wedliny_kielbasa_sucha}   => {wedliny_szynka}    0.07402530 0.4173390  0.1773745 1.1081454  73970
[5]  {wedliny_wedzonki_indyk}   => {wedliny_szynka}    0.04378671 0.3965416  0.1104215 1.0529227  43754
[6]  {}                         => {wedliny_szynka}    0.37661033 0.3766103  1.0000000 1.0000000 376329
[7]  {wedliny_wedzonki_kurczak} => {wedliny_szynka}    0.04101764 0.3702864  0.1107727 0.9832083  40987
[8]  {wedliny_parowki}          => {wedliny_szynka}    0.05859277 0.3661967  0.1600035 0.9723492  58549
[9]  {mieso_wieprzowina}        => {wedliny_szynka}    0.07295650 0.3193632  0.2284436 0.8479938  72902
[10] {mieso_kurczak}            => {wedliny_szynka}    0.07023046 0.3132499  0.2241995 0.8317613  70178
[11] {mieso_kurczak}            => {mieso_wieprzowina} 0.06827600 0.3045324  0.2241995 1.3330744  68225
```

Right hand side elemets of 91% of rules present _wedliny_szynka (ham)_ as the associated product. Only 1 rule indicates pork. Support is quire poor for all pairs, however this is covered by the limitaation set due to dataset characteristics. One should have a look at lift measure where 7 out of 11 rules are better than any randomly chosen rule for a population, so probably only those pairs should be taaken into account later.

**Conclusion:** As mentioned at the beginning, association rules may be useful in inbound promotions/campaaigns a company lacks knwoledge about customers. Leveraging results of the above analysis 7 rules might be implemented in Point of Sales popping up as recommendations for a caashier on what to propose to a customer. Reading through the listed rules, one may say that ham could be proposed to clients who have already added to their baskets cooked meats (wedliny) or cheese, or they could be proposed with pork when they are going to buy chicken meat. 

**Further steps:** Subsequent steps of the analysis are to perform the same type of analysis but at the store level. The results then may be more trustworthy because one cannot now if all the stores have the same stock, hence the rules might be much different and better in terms of support and confidence.
