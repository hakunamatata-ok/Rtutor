---
tags: R & Statistics
---
# 隨機變數 Random Variable
R 有一系列函數可以協助幫忙計算機率統計的值，他們會在那些機率分佈的函數前面加上不同的英文字母來代表不同函數意義。

| 英文字母 | 意義 |
| -------- | -------- |
| ```d```    | density，機率密度  |
| ```p```  | probability，累積機率密度 |
| ```q```  | quantile，分位數  |
| ```r```  | random，隨機 |


| 機率分佈名稱 | 意義 |
| -------- | -------- |
| ```unif```    | 均勻分佈     |
| ```norm```  | 常態分佈  |
| ```binom``` | 二項式分佈 |
| ```pois```  | Possion分佈 |
| ```chisq``` | 卡方分配 |


## Bernoulli 隨機變數與二項隨機變數
### Bernoulli 隨機變數（結果只有二元）
- ```dbinom(x, size, prob)```: $f(x)=P(X=x)$
    - size 實驗次數（當size=1，就是Bernoulli隨機變數）
    - prob X=x的機率
    ```r=
    dbinom(2, 10, 0.7)
    ```
    - plot: 擲硬幣，實驗100次，出現正面為0.5的機率分佈圖
        ```r=
        x <- 1:100
        y <- dbinom(x, size = 100, prob = 0.5)
        plot(x, y, type = "l", ylab = "Probabilty")
        ```
- 對應輸入的二項式分布累積機率值: ```pbinom(q, size, prob)```: $F(x)=P(X \leq x)$
    ```r=
    pbinom(2, 10, 0.7)
    ```
- 對應累積機率值的二項式分布輸入: ```qbinom(p, size, prob)```: $F^{-1}(p)$
    ```r=
    qbinom(0.95, 10, 0.7)
    ```
-  n 個符合二項式分布的隨機值: ```rbinom```
    ```r=
    hist(rbinom(1000, size = 100, prob = 0.5), ylab = "Frequency")
    ```
    ![](https://i.imgur.com/vCrSDpC.png)

- plot of pmf
    ```r=
    # solution 1
    barplot(dbinom(seq(0,15), 15, 0.8), names.arg = seq(0,15))
    # solution 2
    plot(seq(0,15), dbinom(seq(0,15), 15, 0.8), type = "h")
    ```
    - solution1: ![](https://i.imgur.com/eIcglpq.png)
    - solution2: ![](https://i.imgur.com/ii0JDI6.png)

### 二項隨機變數：n個獨立的Bernoulli 隨機變數
- 創造10個相互獨立的Bernoulli(0.5)隨機變數
    ```r=
    rbinom(n = 10, size = 1, prob = 0.5)
    ```
    
## 均勻隨機變數
- 連續型在區間之均勻分佈: $$f(x) = \frac{1}{b-a},\ x \in [a,b]$$
    ![](https://i.imgur.com/i5nkFYA.png)
    - 均勻分佈機率：```dunif(x, min, max)```: $f(x)$
        ```r=
        dunif(2, -5, 5)
        ```
    - plot:
        ```r=
        plot(seq(-0.5, 1.5, by = 0.01), dunif(seq(-0.5, 1.5, by = 0.01), min = 0, max = 1), type = "s")
        ```
        ![](https://i.imgur.com/O8H9dzM.png)

- 分佈函數: $$F(x) =  \left\{  \begin{matrix}
 \begin{matrix}
   0,\ x<a
  \end{matrix}\\
  \begin{matrix}
   \frac{x-a}{b-a},\ a \leq x \leq b
  \end{matrix}\\
  \begin{matrix}
  1,\ x >b
  \end{matrix} 
\end{matrix}
\right\}$$
    

- 均勻分佈累積機率（CDF）：```punif(q, min, max)```: $F(2)=P(X \leq 2)$
    ```r=
    punif(2, -5, 5)
    ```
    - plot:
        ```r=
        plot(seq(-0.5, 1.5, by = 0.01), punif(seq(-0.5, 1.5, by = 0.01), min = 0, max = 1), type = "l")
        ```
        ![](https://i.imgur.com/VwEytGo.png)

- 累積機率值的均勻分布輸入（CDF反函數）：```qunif(p, min, max)```: $F^{-1}(p)$
    ```r=
    qunif(0.7, -5, 5)
    ```
- n 個符合均勻分布的隨機值：```runif(n, min, max)```
    ```r=
     hist(runif(1000), ylab = "Frequency")
    ```
    ![](https://i.imgur.com/MkieMgn.png)

- 創造5個U[-1,1]的均勻隨機變數
    ```r=
    runif(n=5, min = -1, max = 1)
    ```

## 常態分佈
- 常態分佈的機率密度值：```dnorm```
    ```r=
    x <- seq(from = -3, to = 3, by = 0.01)
    y <- dnorm(x)
    plot(x, y, type = "l", ylab = "Density")
    ```
    ![](https://i.imgur.com/m7W2lWA.png)

- 對應輸入的常態分布累積機率值: ```pnorm```
    ```r=
    pnorm(1.96)
    ```
- 對應累積機率值的常態分布輸入: ```qnorm```
    ```r=
    qnorm(0.975)
    ```
-  n 個符合常態分布的隨機值: ```rnorm```
    ```r=
    hist(rnorm(1000), ylab = "Frequency")
    ```
    ![](https://i.imgur.com/gZHoVE1.png)
    
## Poisson分佈
- 單位時間 4 以內某特定事件沒有發生到發生 20 次的機率分布：```dpois```
    ```r=
    x <- 0:20
    y <- dpois(x, lambda = 4)
    plot(x, y, type = "l", ylab = "Probability")
    ```
- 對應輸入的 Poisson 分布累積機率值：```ppois```
    ```r=
    ppois(4, lambda = 4)
    ```
- 對應累積機率值的 Poisson 分布輸入:```qpois```
    ```r=
    qpois(0.62, lambda = 4)
    ``` 
-  n 個符合 Poisson 分布的隨機值：```rpois```
    ```r=
    x <- rpois(1000, lambda = 4)
    hist(x, ylab = "Frequency")
    ```
    ![](https://i.imgur.com/kfWuZbW.png)

## 卡方分配
- 卡方分布的機率密度值：```dchisq()```
    ```r=
    x <- 1:50
    y <- dchisq(x, df = 5)  # df 自由度
    plot(x, y, type = "l", ylab = "Probability")
    ```
- 對應輸入的卡方分布累積機率值：```pchisq()```
    ```r=
    pchisq(5, df = 5)
    ```
- 對應累積機率值的卡方分布輸入：```qchisq()```
    ```r=
    qchisq(0.58, df = 5)
    ```
- n 個符合卡方分布的隨機值：```rchisq()```
    ```r=
    x <- rchisq(1000, df = 5)
    hist(x, ylab = "Frequency")
    ```
    ![](https://i.imgur.com/rqdt0PM.png)


## 練習
1. 有三枚硬幣：公正硬幣、擲出正面機率是 0.7 的硬幣、擲出正面機率是 0.3 的硬幣。投擲這三枚硬幣 100 次，求出現 0 到 100 次正面的機率分布圖。
    :::spoiler
    ```r=
    x <- 0:100
    y1 <- dbinom(x, size = 100, prob = 0.5)
    y2 <- dbinom(x, size = 100, prob = 0.7)
    y3 <- dbinom(x, size = 100, prob = 0.3)
    plot(x, y1, type = "l", ylab = "Probability", ylim = c(0, max(y1, y2, y3)))
    lines(y2, col = "red")
    lines(y3, col = "green")
    ```
    - ```ylim```: y軸的長度設定。
    :::