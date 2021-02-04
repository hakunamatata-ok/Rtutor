---
tags: R & Statistics
---
# Nonlinear Regression
- 前置作業
    ```r=
    library(AER)
    library(stargazer)
    ```
## Data
```r=
data("CASchools")
# add score and STR
CASchools <- CASchools %>% 
    mutate(
      STR = students/teachers,
      score = (read+math)/2
    )
```

## Nonlinear Regression
- 較適合用於探討回歸變量之間的總體關係。e.g. 學校所在區域的人口收入和學生成績之間的關係。
- 有不同形式的非線性回歸：
    - 單一變數非線性回歸：
        - Polynomials：$Y_i = \beta_0+\beta_1*X_1+\beta_r*X^r_i+...+\epsilon_i$
        - Logarithms：$Y_i = \beta_0+\beta_1*ln(X_i)+\epsilon_i$
    - 變數交叉回歸：
        - 判斷變數間交互作用
        - 連續變數和判斷變數交互作用
        - 連續變數間交互作用

## 多項式（Polynomials）模型
- 探討學校所在區域的人口收入和學生成績之間的關係。

### Case 1: 簡單線型回歸（OLS）
- Model：$Score=\beta_0+\beta_1*Income+\epsilon_i$
- 畫圖結果
```r=
library(ggplot2)
ggplot(CASchools, aes(x = income, y = score)) +
  geom_point(colour = "steelblue") +
  geom_smooth(method = "lm", se = F, color = "red") +
  ggtitle(label = "Score vs. District Income and a Linear OLS Regression")
```
![](https://i.imgur.com/DoBEq0C.png)

$\Rightarrow$ 隨著Income增加，score也跟著增加，但可以知道，此線型方程式會有高估的問題，因此不適合解釋學生成績與學區收入的關係。

### Case 2: 二次方回歸（Quadratic Model) 
- Model: $Y_i = \beta_0+\beta_1*income_i+\beta_2*income_i^2+\epsilon_i$
    ```r=
    qm <- lm(score ~ income + I(income^2), data = CAScholls)
    summary(qm)
    ```
    :::danger
    :warning: 
    ```
    lm(score ~ income + income^2, data = CASchools)
    ```
    此結果為 $income$ 對 $score$ 的微分結果，並非 $income$ 和 $income^2$ 對 $score$ 的微分結果。
    :::
- 做假設檢定，判斷 $income^2_i$ 的係數是否為0。
    - $H_0: \beta_2 = 0\ vs.\ H_1: \beta_2 \neq 0$
    - 使用```coeftest()```判斷。
    ```r=
    coeftest(qm, vcov. = vcovHC, type = "HC1")  # reject H0
    ```
- 畫圖
```r=
ggplot(CASchools, aes(x = income, y = score)) +
  geom_point(colour = "steelblue") +
  geom_smooth(method = "lm", se = F, color = "red") +
  geom_smooth(method = "lm", formula = y ~ poly(x,2), 
              se = F, color = "dark orange") +
  ggtitle(label = "Score vs. District Income and a Quadratic Regression")
```
![](https://i.imgur.com/2gAKipC.png)

### Case 3: 立方回歸（Cubic Model）
- Model: $Y_i = \beta_0+\beta_1*income_i+\beta_2*income_i^2+...+\beta_r*X_i^r+\epsilon_i$
    ```r=
    cm <- lm(score ~ poly(income, degree = 3, raw = T),
        data = CASchools)
    summary(cm)
    ```
    - ```raw = T```：設定此預測方程式不為正交多項式。
    :::info
    :information_source: 正交多項式：在連續函數[a,b]所構成的內積空間分成 <f,g>，此兩個區域相加為0。
    $<f,g> = \int_a^bf(x)g(x)w(x)dx=0$ 
    e.g. $<f,g> = \int_{-1}^11*xdx=\frac{x^2}{x}|_{-1}^{1} = 0$
    :::
- 假設檢定，看此方程式的係數，**除了$\beta_1$外**，是否都為0，或是至少一個不為0。
    - $H_0:\beta_2 = ... = \beta_r = 0\ vs.\ H_1: at\ least\ one\ \beta_j \neq 0,\ j=2,...,r$
    - 使用```linearHypothesis()```判斷。
    ```r=
    matrix <- rbind(c(0,0,1,0),c(0,0,0,1))
    linearHypothesis(cm, hypothesis.matrix = matrix, white.adj = "hc1")  # reject H0
    ```
    - ```hypothesis.matrix```是線性限制條件的係數矩陣。
        ![](https://i.imgur.com/yvc5g7B.png)
    $\Rightarrow$ 拒絕$H_0$，表示在這裡的假設中，$\beta_2$和$\beta_3$中至少有一個不為0。

- 再使用```coeftest()```檢定係數。
    ```r=
    coeftest(cm, vcov. = vcovHC, type = "HC1")  # beta3 的p-value接近 0.05，但小於，因此顯著
    ```
    $\Rightarrow$ 拒絕$H_0$，因此確定此方程式可以是 quadratic 或是 cubic。
- 畫圖
```r=
ggplot(CASchools, aes(x = income, y = score)) +
  geom_point(colour = "steelblue") +
  geom_smooth(method = "lm", se = F, color = "red") +
  geom_smooth(method = "lm", formula = y ~ poly(x,2), 
              se = F, color = "orange") +
  geom_smooth(method = "lm", formula = y ~ poly(x,3), 
              se = F, color = "lightcoral") +
  ggtitle(label = "Score vs. District Income and a Cubbic Regression")
```
![](https://i.imgur.com/SQyoc0y.png)

### Quadratic Model在高低Income單位變化的預測結果
- 使用二次方回歸模型，預測收入為10和11時的單位變化，及收入為40和41時的單位變化。
```r=
new_df <- data.frame(income = c(10, 11, 40, 41))
Y_hat_qm <- predict(qm, newdata = new_df)
Y_hat_qm %>% matrix(., nrow = 2, byrow = F) %>%
  {diff(.)}
# 2.962517 0.4240097
```
$\Rightarrow$ 在Quadratic Model中，income越大，單位變化越小。

## 對數（Logarithms）模型
- 看Y或X的變化量的關係，有3種情況：
    - Y受X的變化量影響
    - Y的變化量受X影響
    - Y的變化量受X的變化量影響

### Case 1: Y受X的變化量影響
- Model：$Y_i = \beta_0+\beta_1*ln(X_i)+\epsilon_i$
    ```r=
    logm <- lm(score ~ log(income), data = CASchools)
    summary(logm)
    ```
- 使用`coeftest`進行假設檢定。
    ```r=
    coeftest(logm, vcov = vcovHC, type = "HC1") 
    ```

- 畫圖
```r=
ggplot(CASchools, aes(x = income, y = score)) +
  geom_point(colour = "steelblue") +
  geom_smooth(method = "lm", formula = y ~ log(x), 
              se = F, color = "red") +
  ggtitle(label = "Score vs. District Income and Linear-Log Regression")
```
![](https://i.imgur.com/zOTwGY4.png)

### Case 2: Y的變化量受X影響
- Model：$ln(Y_i) = \beta_0+\beta_1*X_i+\epsilon_i$
    ```r=
    logm2 <- lm(log(score) ~ income, data = CASchools)
    summary(logm2)
    ```
- 使用`coeftest`進行假設檢定。
    ```r=
    coeftest(logm2, vcov = vcovHC, type = "HC1") 
    ```

### Case 3: Y的變化量受X的變化量影響
- Model: $ln(Y_i) = \beta_0+\beta_1*ln(X_i)+\epsilon_i$
    ```r=
    logm3 <- lm(log(score) ~ log(income), data = CASchools)
    summary(logm3)
    ```
- 使用`coeftest`進行假設檢定。
    ```r=
    coeftest(logm3, vcov = vcovHC, type = "HC1") 
    ```
- 畫圖
```r=
ggplot(CASchools, aes(x = income, y = log(score))) +
  geom_point(colour = "steelblue") +
  geom_smooth(method = "lm", se = F, color = "red") +
  geom_smooth(method = "lm", formula = y ~ log(x), 
              se = F, color = "orange") +
  ggtitle(label = "Log-Linear Regression vs. Log-Log Regression")
```
![](https://i.imgur.com/qn39rVM.png)

## 變數交叉回歸
### Case 1: 兩個判斷變數交互作用
- 探討：學校師生比高和英文為第二外語的學生比高，如何影響成績。
    - Model 設定：$Y_i = \beta_0 +\beta_1*D_{1i}+\beta_2*D_{2i}+\beta_3*(D_{1i}*D_{2i})+\epsilon$
        - $D_{1i}$: 判斷師生比，如果$STR \geq 20$，那麼$D_{1i} = 1$；反之，為0。
        - $D_{2i}$: 判斷英文為第二外語的學生比，如果$english \geq 10$，那麼$D_{2i} = 1$；反之，為0。
    ```r=
    # data
    CASchools_bi1 <- CASchools %>%
        mutate(
        D1 = ifelse(STR >= 20, 1, 0),
        D2 = ifelse(english >= 10, 1, 0)
        )
    # model
    bm1 <- lm(score ~ D1*D2, data = CASchools_bi1)
    summary(bm1)
    ```
- 檢定：```coeftest```
    ```r=
    coeftest(bm1, vcov. = vcovHC, type = "HC1")
    ```
- 預測結果
```r=
b_new_df <- data.frame(D1 = c(0,0,1,1), D2 = c(0,1,0,1))
b_new_df$predict <- predict(bm1, b_new_df)
b_new_df
```

### Case 2: 一個連續變數和一個判斷變數交互作用
- 探討：學校師生比和高英文為第二外語的學生比，如何影響成績。
- Model: $Y_i = \beta_0+\beta_1*STR_i+\beta_2*D_{2i}+\beta_2*(STR_i*D_{2i})+\epsilon_i$
    - $D_{2i}$: 英文為第二外語的學生比$\geq 10$ 的判斷係數。
    ```r=
    bcm2 <- lm(score ~ STR + D2 + STR*D2, data = CASchools_bi1)
    summary(bcm2)
    ```
- 檢定：```coeftest```
    ```r=
    coeftest(bcm2, vcov. = vcovHC, type = "HC1")
    ```
- 畫圖
```r=
bcm2_fun0 <- function(x){
  return(coef(bcm2)[1]+coef(bcm2)[2]*x+
    coef(bcm2)[3]*0+coef(bcm2)[4]*0*x)
}

bcm2_fun1 <- function(x){
  return(coef(bcm2)[1]+coef(bcm2)[2]*x+
    coef(bcm2)[3]*1+coef(bcm2)[4]*1*x)
}

ggplot(CASchools_bi1, aes(x = STR, y = score, color = factor(D2))) +
  geom_point() +
  scale_x_continuous(limits = c(0,27), breaks = seq(0,27, by = 5)) +
  stat_function(fun = bcm2_fun0, geom = "line", color = "coral") +
  stat_function(fun = bcm2_fun1, geom = "line", color = "skyblue")
```
![](https://i.imgur.com/uFmrNcv.png)


### Case 3: 兩個連續變數的交互作用
- Model: $Y_i = \beta_0+\beta_1*STR+\beta_2*english+\beta_3*(STR*english)+\epsilon_i$
    ```r=
    ccm3 <- lm(score ~ STR + english + STR * english, data = CASchools_bi1)
    summary(ccm3)
    ```
- 檢定：```coeftest```
    ```r=
    coeftest(ccm3, vcov. = vcovHC, type = "HC1")
    ```
    $\Rightarrow$ Do not reject $H_0: \beta_3 = 0$, $\beta_3$ 對應的p-value = 0.95。