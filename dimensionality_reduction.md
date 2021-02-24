## Principal Component Analysis as a way to get meat stores selling profiles
#### Aleksandra Biłas

### Introduction
Principal Component Analysis usually serves as a method of creating new artificial attributes for further analytical initiatives. While data analysts happen not to look at the new features explainability, the more business-focused people may want to get to know a little bit more about what constitutes the new attributes. Another situation when PCA is helpful is when there is no data dictionaries available or they are too granular. In such a case the algorithm is leveraged in order to find "profiles" of units being analysed. In the following analysis the former approach is applied on meat stores revenue data. Products cateries number over 50 categories and the goal is to find stores' profiles in terms of what type of products they sell most often. PCA method is widely used in both academic and business analyses due to being easy to understand and performant.

## Preliminary analysis
The initial dataset numbers 208 rows (stores) and 52 attributes - ID and 51 attributes for revenue referring to 51 product categories. The first step is to check whethet all columns may be used in terms of variance and null rate. 

```markdown
summary(df)
```
4 columns have been removed due high null rate (greater than 75%). 

Then one needs to make sure that the data meets the PCA assumption about linearity. To assess this scatter plots with linear regression lines were drawn and saved to PDF. Due to big number of variables pairs I attached only few of them as examples.
```markdown
pdf(file = ".../Scatter_plots.pdf", width = 4, height = 4)

# Draw scatter plots along with linear regression line to assess variables linearity
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
![Scatter plot 1: Cielecina vs. Indyk](https://github.com/alebilas/images/blob/main/scatter1.png)

You can use the [editor on GitHub](https://github.com/alebilas/alebilas.github.io/edit/main/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).
