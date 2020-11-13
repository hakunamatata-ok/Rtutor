---
tags: R & Statistics
---
> 本篇參考["*Descriptive statistics in R*", Antoine Soetewey](https://www.statsandr.com/blog/descriptive-statistics-in-r/#data)

# 描述性統計
- 使用data為R內建data：iris
    - 將 iris 命名為 df。
    ```r=
    df <- iris
    df
    ```
### 了解資料
1. `head()`：展示出data的前5行。
    ```r=
    head(df)
    ```
2. `str()`：了解data的結構。
    ```r=
    str(df)
    ```
4. `summary()`：簡單了解data的數值。
    ```r=
    summary(df)
    ```
5. 將df按照"Species"分組，並summary。
    ```r=
    by(df, df$Species, summary)
    ```
6. 另一種看資料描述性統計的方式：
    ```r=
    # install.packages("pastecs")
    library(pastecs)
    stat.desc(df)
    ```
7. 看不同品種的資料量。
    ```r=
    # solution 1
    table(df$Species)
    # solution 2
    summary(df$Species)
    ```
8. 看df有多少行列。
    ```r=
    NROW(df)  # 行數
    NCOL(df)  # 列數
    ```

## 最小值與最大值
- `min()`：找最小值。
    ```r=
    min(df$Sepal.Length)  # 4.3
    ```
- `max()`：找最大值。
    ```r=
    max(df$Sepal.Length)  # 7.9
    ```

- `range`：==呈現==最小值與最大值。
    ```r=
    range(df$Sepal.Length)  # 4.3 7.9
    # first one of the result is minimum, second one of the result is maximum.
    ```
    - ==計算==最小值與最大值的差距。
        ```r=
        max(df$Sepal.Length) - min(df$Sepal.Length)  # 3.6
        ```

## Mean
- 計算平均值
    ```r=
    mean(df$Sepal.Length)
    ```
    - 如果資料有`na`（空值），則可使用`na.rm = T`，跳過該空值進行平均，但可能會有誤差。
        ```r=
        mean(df$Sepal.Length, na.rm = TRUE)
        ```

## Median
- 找出中位數
    ```r=
    median(df$Sepal.Length)
    ```
    - `quantile()`：找出資料中某百分比的數值
        ```r=
        quantile(df$Sepal.Length, 0.05)
        ```

## Quartile
- 四分位數
```r=
# 找出最小值、第一四分位數、中位數、第三四分位數、最大值
quantile(df$Sepal.Length, c(0, 0.25, 0.5, 0.75, 1))
```
- `IQR`：計算第一四分位數與第三四分位數的差距。
    ```r=
    IQR(df$Sepal.Length)
    ```
    ```r=
    quantile(df$Sepal.Length, 0.75) - quantile(df$Sepal.Length, 0.25)
    ```
    
## Standard deviation and variance
- Standard deviation 標準差
    ```r=
    sd(dat$Sepal.Length)
    ```
- Variance 變異數
    ```r=
    var(df$Sepal.Length)
    ```
- 計算data中每個欄位的標準差
    ```r=
    lapply(df[, 1:4], sd)
    ```
    
## Coefficient of variation
- 變異係數
```r=
sd(dat$Sepal.Length) / mean(dat$Sepal.Length)
```

## Contingency table
- 列聯表
- 在此我們看花的大小與各種不同品種的關係。
    1. 假設花萼長度大於其中位數就屬於比較大的花，小於就是比較小的花。
        ```r=
        df$size <- ifelse(df$Sepal.Length) median(df$Sepal.Length), "big", "small")
        
        df
        ```
    2. 看大小花各有多少個
        ```r=
        table(df$size)
        ```
    4. 看各品種花與花大小的列聯表（個數）
        ```r=
        table(df$size, df$Species)
        ```
    5. 看各品種花與花大小的列聯表（比例）
        ```r=
        prop.table(table(df$size, df$Species))
        ```

# 描述性統計畫圖
## Bar plots
1. Species 個數柱狀圖
```r=
# install.packages("ggplot2")
library(ggplot2)
ggplot(df, aes(x = Species)) +
    geom_bar()
```
![barplot1](https://i.imgur.com/fOCahbR.png)

2. Species 個數佔總體比例柱狀圖
```r=
# soluton 1
barplot(prop.table(table(df$Species)))
```
![barplot2](https://i.imgur.com/PcUjBMO.png)

```r=
# solution 2
pt <- prop.table(table(df$Species))
pt <- as.data.frame(pt)
ggplot(pt, aes(x = Var1, y = Freq)) +
  geom_col()
```
![barplot3](https://i.imgur.com/OQEQozG.png)

## Histogram
```r=
# solution 1
hist(df$Sepal.Length)
```
![histogram](https://i.imgur.com/79fECcF.png)

```r=
# solution 2
ggplot(df, aes(x = Sepal.Length)) +
    geom_histogram(bins = 30, color = "black")
```
![histogram2](https://i.imgur.com/ELhwjiQ.png)


## Boxplot
```r=
# solution1
boxplot(df$Sepal.Length ~ df$Species)
```
![](https://i.imgur.com/SpCvnC2.png)

```r=
# solution2
ggplot(df, aes(x = Species, y = Sepal.Length)) +
  geom_boxplot()
```
![](https://i.imgur.com/eubt6mY.png)
