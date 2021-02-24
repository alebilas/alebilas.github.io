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
# Results quality verification
In order to assess quality of obtained components I looked at four aspects - eigenvalues, cumulative variance explained, complexity and uniqueness.

1. Eigenvalues and cumulative variance explained

|            | RC1 | RC4 | RC2 | RC20 | RC13 | RC16 | RC8 | RC3 | RC25 | RC9 | RC18 | RC6 | RC23 | RC11 | RC19 | RC7 | RC14 | RC10 | RC5 | RC15 | RC17 | RC21 | RC30 | RC12 | RC24 | RC29 | RC26 | RC28 | RC27 | RC22 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SS loadings | 13.798 | 1.803 | 1.518 | 1.255 | 1.252 | 1.234 | 1.197 | 1.188 | 1.181 | 1.154 | 1.150 | 1.142 | 1.134 | 1.123 | 1.118 | 1.087 | 1.066 | 1.065 | 1.058. | 1.014 | 1.003 | 0.971 | 0.962 | 0.935 | 0.911 | 0.863 | 0.835 | 0.667 | 0.638 | 0.578 |
| Proportion Var | 0.294 | 0.038 | 0.032 | 0.027 | 0.027 | 0.026 | 0.025 | 0.025 | 0.025 | 0.025 | 0.024 | 0.024 | 0.024 | 0.024 | 0.024 | 0.023 | 0.023 | 0.023 | 0.023. | 0.022 | 0.021 | 0.021 | 0.020 | 0.020 | 0.019 | 0.018 | 0.018 | 0.014 | 0.014 | 0.012 |
| Cumulative Var | 0.294 | 0.332 | 0.364 | 0.391 | 0.418 | 0.444 | 0.469 | 0.495 | 0.520 | 0.544 | 0.569 | 0.593 | 0.617 | 0.641 | 0.665 | 0.688 | 0.711 | 0.733 | 0.756. | 0.777 | 0.799 | 0.819 | 0.840 | 0.860 | 0.879 | 0.897 | 0.915 | 0.929 | 0.943 | 0.955 |

                




3. Cumulative variance
4. Complexity and uniqueness


