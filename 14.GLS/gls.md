---
tags: R & Statistics
---
# GLS
- 前置作業
    - 下載`PoEdata` package
        ```r=
        # install package "PoEdata"
        # install.packages("devtools")
        library(devtools)
        install_git("https://github.com/ccolonescu/PoEdata")
        ```
    - 
    ```r=
    library(dplyr);library(PoEdata)
    library(lmtest);library(broom)
    library(sandwich);library(stargazer)
    ```
    
## Data
- `PoEdata` 中的 `food` data。
    ```r=
    data("food")
    ```
    - `food_exp`：每週食物消費金額
    - `income`：週收入（單位$100）

## Known Form of Variance
- 探討：週收入對每週食物消費金額的影響。
    - 假設一般線性回歸的結果有異方差的問題。</br>
    $\rightarrow$ $exp_i = \beta_0+\beta_1\ income_i+\epsilon_i,\ Var(\epsilon_i)=\sigma_i^2$

    - $Var(\epsilon_i)=\sigma_i^2 = \sigma^2\ income_i^2$
    - 上述模型乘以一個權數$w_i = \frac{1}{income_i}$，得到: $exp_i^* = \beta_1+\beta_0\ income_{i}^*+\epsilon_i^*,\ Var(\epsilon_i^*)=\sigma$
        - $exp^*_i = \frac{exp_i}{income_i}$
        - $x_i^* = \frac{1}{income_i}$
        - $\epsilon_i^* = \frac{\epsilon_i}{income_i}$
    - 此為 weighted least squares，wls 回歸。

1. 設定權數
    ```r=
    w <- 1/food$income
    ```
2. 做wls回歸
    ```r=
    food_wls <- lm(food_exp ~ income, weights = w, data = food)
    ```
3. 做ols回歸
    ```r=
    food_ols <- lm(food_exp ~ income, data = food)
    ```
4. 比較wls回歸和ols回歸結果
    ```r=
    rob_se <- list(sqrt(diag(vcovHC(food_ols, type = "HC1"))),
                   sqrt(diag(vcovHC(food_wls, type = "HC1"))))
    
    stargazer(food_ols, food_wls,
      header = FALSE, 
      type = "text",
      digits = 3, 
      column.labels = c("ols", "wls"),
      dep.var.labels.include = FALSE,
      dep.var.caption = "Model Comparsion: OLS vs. WLS",
      se = rob_se)
    ```
    $\Rightarrow$ wls模型呈現更好的結果，且殘差的標準誤也明顯下降。
    
## Unknown Form of Variance
- Model：$exp_i = \beta_1+\beta_2\ income_{i2}+...+\beta_k\ income_{ik}+\epsilon_i,\ Var(\epsilon_i)=\sigma_i^2=\sigma^2x_i^{\gamma}$
    - 處理殘差：$ln(\hat{\epsilon_i}^2)=\alpha_1+\alpha_2\ log(x_{i2})+...+\alpha_s\ log(x_{is})+v_i$
    - $w_i = \frac{1}{\sigma_i^2}$

1. 計算出ols回歸的殘差平方，並回存至food中，變數名稱為`ehat`。
    ```r=
    food$ehat <- (food_ols$residuals)^2
    ```
2. 做`ehat`對收入的回歸。
    ```r=
    sigmahat_ols <- lm(log(ehat) ~log(income), data = food)
    ```
3. 把`sigmahat_ols`的`fitted.values`取exponential，回存至food中，變數名稱命名為`expo_sigmahat`。
    ```r=
    food$expo_sigmahat <- exp(sigmahat_ols$fitted.values)
    ```
4. 計算權重$w_i$。
    ```r=
    w_sig <- 1/food$expo_sigmahat
    ```
5. 做`food_exp` 對 `income`回歸， 權數為`w_sig`的 fgls 回歸。
    ```r=
    food.fgls <- lm(food_exp~income, weights=w_sig, data=food)
    ```
6. 比較`food_ols` `food_wls` `food_fgls`。
    ```r=
    rob_se <- list(sqrt(diag(vcovHC(food_ols, type = "HC1"))),
          sqrt(diag(vcovHC(food_wls, type = "HC1"))),
          sqrt(diag(vcovHC(food_fgls, type = "HC1"))))

    # generate table
    stargazer(food_ols, food_wls, food_fgls,
      header = FALSE, 
      type = "text",
      digits = 3, 
      column.labels = c("ols", "wls", "fgls"),
      dep.var.labels.include = FALSE,
      dep.var.caption = "Model Comparsion: OLS vs. WLS vs. FGLS",
      se = rob_se)
    ```
    $\Rightarrow$ fgls模型呈現的結果為三者中最好的，且殘差的標準誤也是最小的，與另外兩個模型的差距大。