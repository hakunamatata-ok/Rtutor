---
tags: R & Statistics
---
# 迴歸分析
## 簡單線性回歸 Simple Linear Regression Model
$$Y_i = \alpha + \beta X_i + e_i$$
- $Y_i|X_i \sim (\alpha+\beta X_i, \sigma^2) 相互獨立$
- $e_i$ 是迴歸模型殘差。

### 取期望值
$$E(Y_i|X_i) = \alpha + \beta X_i \\     Var(Y_i|X_I) = \sigma^2 \\ Y_i|X_i \sim (\alpha+\beta X_i, \sigma^2) 相互獨立$$

- 殘差：
    $$E(e_i) = E(X_i e_i) = 0 \\ Var(e_i|X_i)=\sigma^2 \\ Var(X_i e_i)=\sigma^2 E[X_i^2]$$
    
- 共變異數 $Cov(X, e_i) = 0$

### R code
- 以data為R內建data [mtcars](https://stat.ethz.ch/R-manual/R-devel/library/datasets/html/mtcars.html) 為例，探討車子的排氣量（disp）和車子的馬力（hp）的關係。

- 前置作業
    1. data 切割固定
    ```r=
    # 確保data的切割一致
    library(caTools)
    set.seed(123)
    ```
    2. 將mtcars以2:8切割，分別為預測/測試的data，和訓練的data。
    ```r=
    split <- sample.split(mtcars, SplitRatio = 0.8)
    train.df <- subset(mtcars, split == T)    
    predict.df <- subset(mtcars, split == F)
    ```
- 跑回歸
    ```r=
    reg <- lm(formula = hp ~ disp, data = train.df)
    reg
    ```
    - 使用```summary()```來看回歸結果的統計推論。
        ```r=
        summary(reg)
        ```
        - ==變數最後有```***```代表該變數影響最大==
        - $R^2 = 0.6437, Adjusted\ R^2 = 0.6267\ \Rightarrow 解釋程度相對低$
    - 使用```anova()```看變異數分析
        ```r=
        anova(reg)
        ```
    
- 畫圖看回歸結果
    ```r=
    library(ggplot2)
    ggplot(train.df, aes(x = disp, y = hp)) +
      geom_smooth(method = 'lm') +
      geom_point() +
      annotate("text", x = 200, y = 200,
               label = paste0("Y=",round(reg$coefficients[1],2),
                          "+",round(reg$coefficients[2],2),"X")) +
      ggtitle(label = "Simple Linear Regression")
    ```

## 多變量線性迴歸 Multiple Linear Regression 
### 多變數
- data: mtcars
    - 這次看 cyl(汽缸數量)、disp(排氣量)、wt(噸位)、vs(引擎種類，0=自動、1=手動)、am(變數箱，0=自動、1=手動)，對車子馬力（hp）是否能解釋的更清楚。
    
- 設定model
    ```r=
    reg.mult <- lm(hp ~ cyl+disp+wt+vs+am, data = train.df)
    reg.mult
    ```
    ```r=
    summary(reg.mult)
    ```
    ```r=
    anova(reg.mult)
    ```
- 得到的$R^2 = 0.7831, Adjusted\ R^2 = 0.7193\ \Rightarrow 解釋程度有提高，但仍低$

### 除所有hp外的變數帶入，看其他變數對hp的影響。
```r=
reg.all <- lm(hp ~ ., data = train.df)
summary(reg.all)
```
```r=
anova(reg.all)
```
- 得到的$R^2 = 0.9446, Adjusted\ R^2 = 0.8985\ \Rightarrow 解釋程度有提高，但不盡完美$


## 多項式迴歸 Polynomial Regression 
- Model: $Y_i = \alpha + \beta_1 X_i + \beta_2 X_i^2 + ... +\beta_n X_i^n + \varepsilon$

- 針對disp（排放量），生成多個項目，看disp對hp的影響。

- code
    1. 設定變數：假設hp與disp、$disp^2$、$disp^3$有關。
    ```r=
    mtcars2 <- mtcars
    mtcars2$disp2 <- mtcars2$disp^2  # disp平方
    mtcars2$disp3 <- mtcars2$disp^3  # disp立方
    mtcars2
    ```
    2. 拆data：mtcars2
    ```r=
    train.df2 <- subset(mtcars2, split == T)
    predict.df2 <- subset(mtcars2, split == F)
    ```
    3. 回歸
    ```r=
    reg.pn <- lm(hp ~ disp + disp2 + disp3, data = train.df2)
    reg.pn
    ```
    ```r=
    summary(reg.pn)
    ```
    ```r=
    anova(reg.pn)
    ```
    4. 畫圖
    ```r=
    ggplot(train.df2, aes(x = disp, y = hp)) +
  geom_point() +
  geom_line(aes(y = predict(reg.pn, newdata = train.df2)), colour = "blue") +
  annotate("text", x = 156, y = 200,
           label = paste0("Y = ", round(reg.pn$coefficients[1],2), "+",round(reg.pn$coefficients[2],2), "X", round(reg.pn$coefficients[3],2), "X^2",
             "+", round(reg.pn$coefficients[3],2),"X^3"))
    ```
    5. 預測
    ```r=
    predict.df2$predict = predict(reg.pn, newdata = predict.df2)
    predict.df2
    ```
    
## 標準化回歸 Data Scaling + Mutiple Regression
- 找到$\hat{\beta_i}$來看變動一單位的狀況。
- code
    1. 資料標準化
        ```r=
        # 原始資料
        summary(mtcars)
        
        # 標準化
        mtcars.scale <- scale(mtcars)
        summary(mtcars.scale)
        ```
    2. 標準化回歸
        ```r=
        reg.scale <- lm(hp ~ ., data = as.data.frame(mtcars.scale))
        summary(reg.scale)
        ```
        - $R^2 = 0.9028, Adjusted\ R^2 = 0.8565$, 比沒有標準化的回歸結構要好。
    3. 比較標準化前後回歸係數差異
        ```r=
        reg.all$coefficients
        reg.scale$coefficients
        ```

    
## 羅吉斯回歸 Logistic Regression
- 判斷==二元的類別變項==
- 這邊一樣以mtcars為例，但這次判斷vs(引擎種類，0=自動、1=手動)。
- code
    ```r=
    greg.s <- glm(vs ~ ., data = train.df2, family = 'binomial')
    greg.s
    ```
    ```r=
    summary(greg.s)
    ```
    
## 練習
1. 請使用iris（R Studio 內建data），找最能解釋花瓣長（Petal.Length)的變數。
    :::spoiler
    ```r=
    reg.iris <- lm(Petal.Length ~ ., data = iris)
    summary(reg.iris)
    ```
    - Sepal.Length, Petal.Width, Species最能詮釋花瓣長度不同的原因。
    - 解釋程度：Multiple R-squared:  0.9786,	Adjusted R-squared:  0.9778 
    :::
    
2. 一樣使用iris，但取出Species符合 setosa、versicolor的資料$^1$。將資料2:8拆開，利用 Sepal.Length, Sepal.Width, Petal.Width, Petal.Length 來判斷該花屬於哪種品種。
    - 1. 可使用以下code來取出需要的資料：
    ```r=
    iris2 <- subset(iris, Species != "virginica")
    iris2$Species.type <- ifelse(iris2$Species == "setosa", 0, 1)
    ```
    :::spoiler
    1. 分開資料
        ```r=
        set.seed(123)
        split2 <- sample.split(iris2, SplitRatio = 0.8)
        train.ir2 <- subset(iris2, split2 == T)
        predict.ir2 <- subset(iris2, split2 == F)
        ```
    2. 使用train.ir2訓練logistic模型
        ```r=
        greg.ir2 <- glm(Species.type ~ ., data = train.ir2, family = "binomial")
        summary(greg.ir2)
        ```
    3. 使用預測資料來和訓練好的模型來預測Species。
        ```r=
        predict.ir2$Predict.Species.type <-    predict(greg.ir2, newdata = predict.ir2)
        predict.ir2$Predict.Species <- ifelse(predict.ir2$Predict.Species.type < 0, "setosa", "versicolor")

        mean(predict.ir2$Species == predict.ir2$Predict.Species)  # 1
        ```
        - 因為 mean(predict.ir2$Species == predict.ir2$Predict.Species) = 1，所以都正確
    :::