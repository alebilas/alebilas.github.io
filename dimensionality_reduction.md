## Principal Component Analysis as a way to get meat stores selling profiles
#### Aleksandra Biłas

### Introduction
Principal Component Analysis usually serves as a method of creating new artificial attributes for further analytical initiatives. While data analysts happen not to look at the new features explainability, the more business-focused people may want to get to know a little bit more about what constitutes the new attributes. Another situation when PCA is helpful is when there is no data dictionaries available or they are too granular. In such a case the algorithm is leveraged in order to find "profiles" of units being analysed. In the following analysis the former approach is applied on meat stores revenue data. Products cateries number over 50 categories and the goal is to find stores' profiles in terms of what type of products they sell most often. PCA method is widely used in both academic and business analyses due to being easy to understand and performant.

## Required packages
```markdown
library(caret)
library(psych)
library(maptools)
```

## Preliminary analysis
The initial dataset numbers 208 rows (stores) and 52 attributes - ID and 51 attributes for revenue referring to 51 product categories. The first step is to check whethet all columns may be used in terms of variance and null rate. 

```markdown
summary(df)
```
4 columns have been removed due high null rate (greater than 75%). 

Then one needs to make sure that the data meets the PCA assumption about linearity. To assess this scatter plots with linear regression lines were drawn and saved to PDF. Due to big number of variables pairs I attached only few of them as examples.
```markdown
pdf(file = ".../Scatter_plots.pdf", width = 4, height = 4)

for(i in 2:ncol(df)) {
  for(j in 2:ncol(df)) {
    x <- df[, i]
    y <- df[, j]
    plot(x, y,
         xlab = colnames(df[i]), ylab = colnames(df[j]),
         pch = 19, frame = FALSE)
    abline(lm(y ~ x, data = df), col = "blue")
  }
}

dev.off()
```
![Scatter plot](https://github.com/alebilas/images/blob/main/scatter.png)
All attributes show linearity so there is no need to look for help in any spectral methods such as Kernel PCA.

# Analysis
First data needs to be scaled in order to equilibrate the magnitude of the variables, simultaneously ensuring outliers to have reduced their influence (for instance Min Max Scaler does not provide this).

```markdown
df_preproc <- preProcess(df, method=c("center", "scale"))
df.s <- predict(df_preproc, df)
```

## Principal Components Analysis
I used _psych_ library PCA algorithm as it enables usage of rotation. Rotations are used to standardize loadings and make them more interpretable.

```markdown
pca = principal(df.s, nfactors=30, rotate="varimax")
pca
```

## Results quality verification
In order to assess quality of obtained components I looked at four aspects - eigenvalues (SS loadings), cumulative variance explained, complexity and uniqueness.

_Eigenvalues and cumulative variance explained_

|            | RC1 | RC4 | RC2 | RC20 | RC13 | RC16 | RC8 | RC3 | RC25 | RC9 | RC18 | RC6 | RC23 | RC11 | RC19 | RC7 | RC14 | RC10 | RC5 | RC15 | RC17 | RC21 | RC30 | RC12 | RC24 | RC29 | RC26 | RC28 | RC27 | RC22 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SS loadings | 13.798 | 1.803 | 1.518 | 1.255 | 1.252 | 1.234 | 1.197 | 1.188 | 1.181 | 1.154 | 1.150 | 1.142 | 1.134 | 1.123 | 1.118 | 1.087 | 1.066 | 1.065 | 1.058. | 1.014 | 1.003 | 0.971 | 0.962 | 0.935 | 0.911 | 0.863 | 0.835 | 0.667 | 0.638 | 0.578 |
| Proportion Var | 0.294 | 0.038 | 0.032 | 0.027 | 0.027 | 0.026 | 0.025 | 0.025 | 0.025 | 0.025 | 0.024 | 0.024 | 0.024 | 0.024 | 0.024 | 0.023 | 0.023 | 0.023 | 0.023. | 0.022 | 0.021 | 0.021 | 0.020 | 0.020 | 0.019 | 0.018 | 0.018 | 0.014 | 0.014 | 0.012 |
| Cumulative Var | 0.294 | 0.332 | 0.364 | 0.391 | 0.418 | 0.444 | 0.469 | 0.495 | 0.520 | 0.544 | 0.569 | 0.593 | 0.617 | 0.641 | 0.665 | 0.688 | 0.711 | 0.733 | 0.756. | 0.777 | 0.799 | 0.819 | 0.840 | 0.860 | 0.879 | 0.897 | 0.915 | 0.929 | 0.943 | 0.955 |

```markdown
scree(pca$loading)
```
![Eigenvalues, scree plot](https://github.com/alebilas/images/blob/main/eigenvalues.png)
In the table one can see that there are 21 rotated components with eigenvalues greater than 1 meaning that the linear combination of RCs components explains more variance than single attributes. However when we look at cumulative variance, those RCs explain 78% of variance which is too little to proceed. As we do want to keep components that explain as much variance as possible (at least 95%), all resulting RCs should be kept.              

_Complexity and uniqueness_

Uniqueness of a variable refers to variance that is unique to the variable and not shared with others. The lower its value is, the higher relevance of this variable.
Complexity represents a number of latent components needed to account for an attribute. An ideal value of complexity is 1 meaning that each attribute would constitute only one component.

```markdown
comp_uniq = data.frame(Complexity=pca$complexity, Uniqueness=pca$uniqueness)
comp_uniq
```

| | Complexity | Uniqueness |
| --- | --- | --- |
| Boczki.wedzone                  | 1.879905 | 0.103943130 |
| Cielecina                       | 1.943003 | 0.035199734 |
| Dagoma                          | 1.501562 | 0.006662387 |
| Elementy.z.kaczki               | 2.471706 | 0.030501360 |
| Flaki                           | 1.743190 | 0.018213819 |
| Fungopol                        | 1.271656 | 0.007411272 |
| Galarety                        | 2.784469 | 0.038515497 |
| Indyk                          | 11.184126 | 0.084890436 |
| Kabanosy                        | 2.214951 | 0.075323615 |
| Kacik.Smaku                     | 1.549833 | 0.006200366 |
| Kaczka                          | 1.736963 | 0.005834061 |
| Kamis                           | 2.530438 | 0.025522958 |
| Kanapkowe.drobiowe              | 1.513890 | 0.092984734 |
| Kanapkowe.parzone               | 1.544458 | 0.080752139 |
| Kanapkowe.suche                 | 1.732165 | 0.085350909 |
| Kosci.wedzone                   | 2.931963 | 0.045473343 |
| Kowalewski                      | 2.023866 | 0.027673572 |
| Krokiety..nalesniki.paszteciki  | 1.475822 | 0.025626192 |
| Krokus                          | 3.083474 | 0.064745493 |
| Kurczak                         | 2.558227 | 0.055833977 |
| Laskowe.parzone                 | 1.388707 | 0.080477541 |
| Laskowe.suche                   | 1.496320 | 0.002842708 |
| Marynaty                        | 1.112299 | 0.003609784 |
| Maslo.smalec                    | 1.793405 | 0.021641914 |
| Metki                           | 1.732505 | 0.078480324 |
| Miesa.garmazeryjne              | 2.261448 | 0.015488995 |
| Miesa.i.Garmazerka.z.pieca      | 1.291528 | 0.006524275 |
| Olej..oliwy                     | 1.381727 | 0.013810775 |
| Parowki                         | 1.430763 | 0.070155410 |
| Pierogi..uszka                  | 3.037208 | 0.031626271 |
| Podravka                        | 2.040404 | 0.047313026 |
| Podrobowe                       | 1.362268 | 0.048049831 |
| Pozostala.Garmazerka            | 1.638206 | 0.013136803 |
| Pozostale.Dodatki               | 1.528274 | 0.017368880 |
| Pozostale.Miesa                 | 1.381097 | 0.014109227 |
| Produkty.maczne                 | 2.246241 | 0.027724080 |
| Proeko                          | 1.360767 | 0.010666520 |
| Rozne                           | 1.262522 | 0.059353383 |
| Salami                          | 1.963190 | 0.078427021 |
| Sery                            | 4.501061 | 0.071552006 |
| Soki..napoje                    | 1.679097 | 0.063547946 |
| Surowe                          | 2.214241 | 0.103279393 |
| Surowki                         | 1.497082 | 0.017624678 |
| Wedzonki                        | 1.574704 | 0.054460315 |
| Wedzonki.drobiowe               | 1.439354 | 0.075711444 |
| Wieprzowina                     | 2.422077 | 0.073649189 |
| Wolowina                        | 5.880608 | 0.085016460 |

To list attributes that may be considered to be removed the following conditions were used and returned _Wolowina_ and _Indyk_ items.:
```markdown
worst = comp_uniq[comp_uniq$Complexity>5 | comp_uniq$Uniqueness>0.5,]
worst
```

Mean item complexity is 2.2 which is means that on average one component is loaded by 2 items. Removing _Wolowina_ and _Indyk_ from the dataset resulted in lowering the mean item complexity to 1.9, however those attributes are very significant business-wise, hence I decided not to remove them.

## Rotated loading interpretation
Now when one knows that obtained components are of good quality, there is time for interpretation. To see only significant loadings I selected only those that are higher than 0.5. Some RCs does not show any significancy in terms of single items so the interpretation will be done only for those components that comprise of at least one significant loading.

```markdown
print(loadings(pca), digits=3, cutoff=0.5, sort=TRUE)
```

|                                 | RC1   | RC4   |   RC2 |   RC20|   RC13|   RC16|   RC8 |   RC3 |   RC25|   RC9 |   RC18|   RC6 |   RC23|   RC11| RC19  | RC7    | RC14  | RC10  | RC5  |  RC15 |  RC17 |  RC21 |  RC30  | RC12 |  RC24  | RC29  | RC26 |  RC28 | RC27 |  RC22 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Boczki.wedzone                  | 0.806 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                                     
| Kabanosy                        | 0.785 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                                     
| Kanapkowe.drobiowe              | 0.855 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                              
| Kanapkowe.parzone               | 0.859 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                              
| Kanapkowe.suche                 | 0.831 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                              
| Kosci.wedzone                   | 0.654 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       | 0.595 |                                                                                 
| Kurczak                         | 0.752 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                               
| Laskowe.parzone                 | 0.883 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                                 
| Metki                           | 0.835 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                                     
| Parowki                         | 0.881 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                                     
| Podrobowe                       | 0.902 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                                     
| Salami                          | 0.809 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                                     
| Sery                            | 0.638 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                             
| Surowe                          | 0.772 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                               
| Wedzonki                        | 0.867 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                               
| Wedzonki.drobiowe               | 0.877 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                               
| Wieprzowina                     | 0.765 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                               
| Rozne                           |       | 0.915 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                              
| Soki..napoje                    |       | 0.848 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                              
| Cielecina                       |       |       |  0.827|       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                      
| Wolowina                        |       |       |  0.559|       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                                      
| Pozostale.Miesa                 |       |       |       | 0.915 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                               
| Proeko                          |       |       |       |       | 0.920 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                        
| Krokiety..nalesniki.paszteciki  |       |       |       |       |       | 0.893 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                                 
| Olej..oliwy                     |       |       |       |       |       |       | 0.915 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                          
| Dagoma                          |       |       |       |       |       |       |       | 0.898 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                                   
| Podravka                        |       |       |       |       |       |       |       |       | 0.806 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                            
| Fungopol                        |       |       |       |       |       |       |       |       |       | 0.938 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |                     
| Kacik.Smaku                     |       |       |       |       |       |       |       |       |       |       | 0.891 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |              
| Surowki                         |       |       |       |       |       |       |       |       |       |       |       | 0.894 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       
| Krokus                          |       |       |       |       |       |       |       |       |       |       |       |       | 0.719 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       | 
| Miesa.i.Garmazerka.z.pieca      |       |       |       |       |       |       |       |       |       |       |       |       |       | 0.934 |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       | 
| Pozostala.Garmazerka            |       |       |       |       |       |       |       |       |       |       |       |       |       |       | 0.876 |       |       |       |       |       |       |       |       |       |       |       |       |       |       | 

## Naming the components
**RC1**: Items used on a daily basis. Products for breakfast, lunch.
**RC4**: Juices, sweets.
**RC2**: Beef meat.
**RC20**: Other meat, usually used for soups.
**RC13, RC3, RC9, RC23**: Meal sides, conserves.
**RC16, RC11, RC19**: Ready-made food.
**RC8**: Oils.
**RC20, RC18**: Seasonings.
**RC6**: Raw salads.
**RC22**: Smoked bones.

The obtained components may be used as inputs in further analyses, for instance in clustering algorithms.
