---
tags: R & Statistics
---
# Model Specification
- 前置作業
    ```r=
    # install.packages("AER")
    library(AER)
    library(dplyr)
    ```

## Data
```r=
data("CASchools")
```
- 此資料為1998～1999年，加州內兩個地區共420所學校，針對5年級學生做評量的數據。共14個變數：
    - ```district```: 學校區域編碼
    - ```school```: 學校名稱
    - ```county```: 所屬城鎮
    - ```grades```: 學校所屬地區
    - ```students```: 學生入學率
    - ```teachers```: 老師人數
    - ```calworks```: 公共援助計劃的學生百分比
    - ```lunch```: 有資格享受低價午餐的學生百分比
    - ```computer```: 學校電腦數量
    - ```expenditure```: 學校在每位學生的支出
    - ```income```: 所屬地區人口平均收入
    - ```english```: 英文為第二外語的學生百分比
    - ```read```: 學生平均閱讀成績
    - ```math```: 學生平均數學成績
- 生成兩個變數：師生比```STR```和平均成績```score```。
    :::spoiler
    ```r=
    CASchools <- CASchools %>% 
        mutate(
          STR = students/teachers,
          score = (read+math)/2
        )
    ```
    :::

## 分成
- 探討學生的平均成績會受到學校師生比變化怎樣的影響？其因果關係為何？

### 1. 先了解```score```和學生特徵變數（english, calworks, lunch）的關係
```r=
# set up arrangement of plots
p <- rbind(c(1, 2), c(3, 0))
graphics::layout(mat = p)

plot(score ~ english, 
     data = CASchools, 
     col = "steelblue", 
     pch = 20, 
     xlim = c(0, 100),
     cex.main = 0.9,  # 標題放大倍數
     main = "Percentage of English language learners")

plot(score ~ lunch, 
   data = CASchools, 
   col = "steelblue", 
   pch = 20,
   cex.main = 0.9,  
   main = "Percentage qualifying for reduced price lunch")

plot(score ~ calworks, 
   data = CASchools, 
   col = "steelblue", 
   pch = 20, 
   xlim = c(0, 100),
   cex.main = 0.9,
   main = "Percentage qualifying for income assistance")
```
![](https://i.imgur.com/onz5O0a.png)

### 2. 看```score```和學生特徵變數的相關係數
```r=
# score vs english
cor(CASchools$score, CASchools$english)

# score vs lunch
cor(CASchools$score, CASchools$lunch)
    
# score vs calworks
cor(CASchools$score, CASchools$calworks)
```

### 3. 判斷各個學生特徵變數間，是否存在共線性
- Solution 1: 如果**相關係數接近0.8**，則可能有共線性的問題。
    ```r=
    # english vs lunch
    cor(CASchools$english,     CASchools$lunch)

    # english vs calworks
    cor(CASchools$english, CASchools$calworks)

    # lunch vs calworks
    cor(CASchools$lunch, CASchools$calworks)
    ```
- Solution 2: 將```score```對```english```, ```lunch```, ```calworks```, ```STR```，進行回歸分析，並使用```car```套件下的```vif()```來看變異數膨脹因素（VIF, variance inflation faction）。
    - VIF 越小，共線性的機率也越小。
    - 當 **VIF >> 10**，則有嚴重的共線性。
    ```r=
    m <- lm(score ~ STR+calworks+lunch+english, data = CASchools)
    car::vif(m, digits = 3)
    ```

### 4. 設計Model
- 有5種設定方式：
    1. $Score = \beta_0+\beta_1*STR+\epsilon$
    2. $Score = \beta_0+\beta_1*STR+\beta_2*english+\epsilon$
    3. $Score = \beta_0+\beta_1*STR+\beta_2*english+\beta_3*lunch+\epsilon$
    4. $Score = \beta_0+\beta_1*STR+\beta_2*english+\beta_4*calworks+\epsilon$
    5. $Score = \beta_0+\beta_1*STR+\beta_2*english+\beta_3*lunch+\beta_4*calworks+\epsilon$

```r=
# 設定5種model
m1 <- lm(score ~ STR, data = CASchools)
m2 <- lm(score ~ STR + english, data = CASchools)
m3 <- lm(score ~ STR + english + lunch, data = CASchools)
m4 <- lm(score ~ STR + english + calworks, data = CASchools)
m5 <- lm(score ~ STR + english + lunch + calworks, data = CASchools)
```

### 5. 選擇適合的Model
1. 把所有模型結果合在一起看。
    ```r=
    library(stargazer)
    stargazer(m1, m2, m3, m4, m5, digits = 3, type = "text",
              column.labels = c("m1", "m2", "m3", "m4", "m5"))
    ```
    ![](https://i.imgur.com/KR5P0hR.png)
    - Model 3 和 Model 5 的 $R^2$ 和 $Adjusted\ R^2$ 最大，因此解釋程度最大。
    - Model 5 中 ```calworks```影響的不顯著，因此相較之下，Model 3 略勝一籌。

3. 透過判斷 AIC 和 BIC 來選擇
    - AIC 和 BIC 越小，代表Model越好。
    ```r=
    ab <- broom::glance(m1) %>% 
      tibble::add_column(., model = "STR", .before = "r.squared")

    ab <- broom::glance(m2) %>% 
      tibble::add_column(., model = "STR + english", .before = "r.squared") %>%
      rbind(ab,.)

    ab <- broom::glance(m3) %>% 
      tibble::add_column(., model = "STR + english + lunch", 
                         .before = "r.squared") %>%
      rbind(ab,.)

    ab <- broom::glance(m4) %>% 
      tibble::add_column(., model = "STR + english + calworks", 
                         .before = "r.squared") %>%
      rbind(ab,.)

    ab <- broom::glance(m5) %>% 
      tibble::add_column(., model = "STR + english + lunch + calworks", 
                         .before = "r.squared") %>%
      rbind(ab,.)

    ab
    ```
    ![](https://i.imgur.com/kSwUQZf.png)
    - 比較 AIC & BIC 後，可發現 Model 3的最小，因此 Model 3 較適合解釋。

$\Rightarrow$ 綜上所述，我們選擇 Model 3: $Score = \beta_0+\beta_1*STR+\beta_2*english+\beta_3*lunch+\epsilon$