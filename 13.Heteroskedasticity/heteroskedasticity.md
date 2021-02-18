---
tags: R & Statistics
---
# Heteroskedasticity
- 前置作業
    ```r=
    library(dplyr);library(PoEdata)
    library(lmtest);library(broom)
    library(sandwich);library(stargazer)
    library(ggplot2)
    ```

## Data
### `food`
- `PoEdata` 中的 `food` data。
    ```r=
    data("food")
    ```
    - `food_exp`：每週食物消費金額
    - `income`：週收入（單位$100）
- Graphs
    - data
        ```r=
        ggplot(food, aes(x = income, y = food_exp)) +
      geom_point(color = "dark blue") +
      geom_smooth(formula = y~x, method = "lm", se = F)
        ```
    - `food_exp`對`income`做ols回歸的殘差和`income`之間的關係
        ```r=
        food_ols <- lm(food_exp ~ income, data = food)
        food <- food %>%
          mutate(res_ols = food_ols$residuals)

        ggplot(food, aes(x = income, y = res_ols)) +
          geom_point(color = "dark blue")
        ```
        $\rightarrow$ 殘差最大最小值差距大。

### `cps2`
- `PoEdata` 中的 `cps2` data。
    ```r=
    data("cps2")
    ```
    - `wage`: 時薪
    - `educ`: 受教育年數
    - `exper`: 經歷
    - `black`: 是否為黑人
    - `married`: 是否已婚
    - `union`: 是否為工會成員
    - `south`: 是否是南方人
    - `fulltime`: 是否為全職員工
    - `metro`: 是否住在都會區
        
## Heteroskedasticity Tests
- $y_i = \beta_1+\beta_{2}x_{i2}+...+\beta_kx_{ik}+e_i$
    - $var(y_i)=E(e_i^2)=\alpha_1+\alpha_2z_{i2}+...+\alpha_sz_{is}$

### The Breusch-Pagan Test
- $H_0:\ \alpha_1 = \alpha_2 = ...=\alpha_s = 0$
    - $\hat{e}^2_i = \alpha_1+\alpha_2z_{i2}+...+\alpha_sz_{is}+v_i$

- Test statistics: $\chi^2 = N*R^2 ~ \chi^2_{(s-1)}$
    - $N$: the number of 'observations' from a model fit. 
    - $R^2$: 殘差回歸的一種結果。
    - $S$: $\beta$ 在模型裡的個數。
- 使用data：`food`

#### solution 1
1. 做`food_exp` 對 `income` 回歸
    ```r=
    model1 <- lm(food_exp~income, data=food)
    ```
2. 將`model1`的殘差做平方，存回food中，變數取名為`res_sqr`。
    ```r=
    food$res_sqr <- (model1$residuals)^2
    ```
3. 建立殘差的回歸等式。
    ```r=
    model1_res <- lm(res_sqr ~ income, data=food)
    stat_model1_res <- glance(model1_res)
    ```
4. 找到 $\alpha = 0.05$時，卡方分配的 critical value。
    ```r=
    chisq_cr <- qchisq(1-alpha, stat_model1_res$df)
    chisq_cr
    ```
5. 計算該模型的卡方分配結果及其p-value。
    ```r=
    # 卡方分配結果
    chisq <- nobs(model1_res)*stat_model1_res$r.squared
    chisq
    
    # p-value
    p_value <- 1 - pchisq(chisq, stat_model1_res$df)
    p_value
    ```
    $\rightarrow$ $\because \chi^2 = 7.38 > \chi^2_{cr} = 3.84,\ p-value = 0.0066 < 0.05$，$\therefore$ 拒絕虛無假設，殘差有 Heteroskedasticity 問題。
    
#### solution 2
- 使用`lmtest` package的 `bptest` function，可以做到上述結果。
    ```r=
    bptest(model1)
    ```

### The Goldfeld-Quandt Test
-  依照判斷變量（不是等於0，就是等於1），分別做回歸，判斷這兩個模型的標準差是否相同。
-  $H_0:\ \hat{\sigma_1}^2 = \hat{\sigma_0}^2$
-  統計量：$F = \frac{\hat{\sigma_1}^2}{\hat{\sigma_0}^2} ～ F_{(N_1-k, N_0-k)}$
#### solution 1

#### 使用data：`cps2`
1. 設定model：$wage = \beta_1+\beta_2\ educ+\beta_3\ exper+\beta_4\ metro+e$
    - 依照`metro`分別設定。
    ```r=
    in_metro <- filter(cps2, metro == 1)
    in_country <- filter(cps2, metro == 0)
    
    # model
    wg1 <- lm(wage ~ educ + exper, data = in_metro)
    wg0 <- lm(wage ~ educ + exper, data = in_country)
    ```
2. $\alpha = 0.05$下，F分配的 critical value。
    ```r=
    df1 <- wg1$df.residual
    df0 <- wg0$df.residual
    
    # critical value
    F_lc <- qf(0.05/2, df1, df0)  # left
    F_rc <- qf(1 - (0.05/2), df1, df0)  # right
    F_lc
    F_rc
    ```
3. 計算這兩個模型的F值。
    ```r=
    F_stat <- glance(wg1)$sigma^2/glance(wg0)$sigma^2
    F_stat
    
    # p-value
    1-pf(F_stat, df1, df0)
    ```
    $\rightarrow$ $\because\ F = 2.09 > Right Critical Value = 1.26$, $\therefore$ 拒絕虛無假設，殘差有 Heteroskedasticity 問題。
    
#### 使用data：`food`
1. 將資料分成低收入與高收入。（小於收入中位數為低收入，大於則為高收入。）並依照收入高低做不同的回歸模型。
    - $H_0:\ \hat{\sigma_{hI}}^2 \leq \hat{\sigma_{lI}}^2$
    ```r=
    low_I <- filter(food, income <= median(income))
    high_I <- filter(food, income >= median(income))

    # model
    I_l <- lm(food_exp~income, data=low_I)
    I_h <- lm(food_exp~income, data=high_I)
    ```
2. $\alpha = 0.05$下，F分配的 critical value。
    ```r=
    df_l <- I_l$df.residual
    df_h <- I_h$df.residual
    
    # critical value
    F_c <- qf(1 - (0.05/2), df_h, df_l)
    F_c
    ```
3. 計算這兩個模型的F值及其p-value。
    ```r=
    F_stat2 <- glance(I_h)$sigma^2/glance(I_l)$sigma^2
    F_stat2
    
    # p-value
    1-pf(F_stat2, df_h, df_l)
    ```
    $\rightarrow$ $\because\ F = 3.61 > Right Critical Value = 2.6$, $\therefore$ 拒絕虛無假設，殘差有 Heteroskedasticity 問題。

#### solution 2
- 使用`lmtest` package的 `gqtest` function，可以做到上述結果。
    ```r=
    foodols <- lm(food_exp ~ income, data = food)
    gqtest(foodols, point=0.5, alternative="greater", order.by=food$income)
    ```