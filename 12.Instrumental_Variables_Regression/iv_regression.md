---
tags: R & Statistics
---
# Instrumental Variables Regression
- 前置作業：
    ```r=
    library(AER)
    library(stargazer)
    library(dplyr)
    ```
## Data
- AER package 內的資料 —— CigarettesSW。
- 記錄1985～1995年，美國48個州，香菸的消費資訊。是一個 panel data。 
    ```r=
    data("CigarettesSW")
    ```
    - state：州.
    - year：會計年度
    - cpi：消費指數
    - population：州人口
    - packs：人均煙銷售量
    - income：州個人名目所得
    - tax：各會計年度下，州平均的聯邦稅和當地的平均消費稅
    - price：各會計年度下，平均價格，包含消費稅
    - taxs：各會計年度下，州平均消費稅，包含營業稅

## Model
### A Single Regressor and A Single Instrument
$$log(Q^{cigarettes}_i)=\beta_0+\beta_1log(P^{cigarettes}_i)+u_i$$

- $Q^{cigarettes}_i$：第i州人均煙銷售量
- $P^{cigarettes}_i$：第i州稅後實質價格
    - $P^{cigarettes}_i$ 與 $SalesTax$ 有關。
    $\Rightarrow$ $log(P^{cigarettes}_i) = \pi_0+\pi_1SalesTax_i+v_i$

1. 生成人均實質價格和營業稅的變數：
    ```r=
    # generate 2 variables
    CigarettesSW <- CigarettesSW %>%
        mutate(
          Rprice = price/cpi,
          SalesTax = (taxs - tax)/cpi
        )
        
    # correlation between SalesTax and price
    cor(CigarettesSW$SalesTax, CigarettesSW$price)
    ```
    - 人均實質價格 ```Rprice```
    - 營業稅 ```SalesTax```
2. 取出1995年的資料來分析：
    ```r=
    df1995 <- subset(CigarettesSW, year == 1995)
    ```
3. 跑 $log(P^{cigarettes}_i)$ 對 $SalesTax$ 的回歸
    ```r=
    # first-stage regression
    cig_s1 <- lm(log(Rprice) ~ SalesTax, data = df1995)
    
    coeftest(cig_s1, vcov = vcovHC, type = "HC1")
    ```
    $\Rightarrow$ $log(\hat{P}^{cigarettes}_i) = 4.62+0.031SalesTax_i$
    ```r=
    summary(cig_s1)
    ```
4. 存取第一階段回歸，$log(\hat{P}^{cigarettes}_i)$ 的結果
    ```r=
    cigp_pred1 <- cig_s1$fitted.values
    ```
5. 做第二階段，$log(Q^{cigarettes}_i)$ 對 $log(\hat{P}^{cigarettes}_i)$ 回歸:
    - 方法一：直接回歸
    ```r=
    cig_s2 <- lm(log(df1995$packs) ~ cigp_pred1)
    
    coeftest(cig_s2, vcov = vcovHC, type = "HC1")
    ```
    $\Rightarrow$ $log(\hat{Q}^{cigarettes}_i)=9.72+1.08\ log(P^{cigarettes}_i)$
    - 方法二：使用`ivreg()`來做回歸，此function自動處理處理成TSLS。
    ```r=
    cig_ivreg <- ivreg(log(packs) ~ log(Rprice) | SalesTax, data = df1995)
    
    coeftest(cig_ivreg, vcov = vcovHC, type = "HC1")
    ```
    - 兩者得到的係數結果一致，但標準差有差。
6. 根據5.的回歸結果，我們可以猜測價格增加1%，香菸的消費量會掉1.08%。$\Rightarrow$ 有彈性。

### The General IV Regression Model
$$log(Q^{cigarettes}_i)=\beta_0+\beta_1\ log(P^{cigarettes}_i)+\beta_2\ log(Income_i)+u_i$$
- $Income_i$ 是香菸需求函數的**外生變數**，實質收入。
    - $E(u_i|Income_i) = 0$

1. 生成實質人均所得
    ```r=
    df1995$Rincome <- with(df1995, income/population/cpi)
    ```
2. 使用`ivreg()`得到係數。
    ```r=
    cig_ivreg2 <- ivreg(log(packs) ~ log(Rprice) + log(Rincome) | log(Rincome) + SalesTax, data = df1995)
    
    coeftest(cig_ivreg2, vcov = vcovHC, type = "HC1")
    ```
    $\Rightarrow$ $log(\hat{Q}^{cigarettes}_i) = 9.42-1.14\ log(\hat{P}^{cigarettes}_i)+0.21\ log(income_i)$
    
## Check Instrument Validity
- 估計香菸長期的需求彈性——看1985～1995年的資料。
    $$log(Q^{cigarettes}_i)=\beta_0+\beta_1\ [log(P^{cigarettes}_{i,1995})-log(P^{cigarettes}_{i,1985})]+\\ \beta_2\ [log(Income_{i,1995})-log(Income_{i,1985})]+u_i$$
    
1. 重新整理 CigarettesSW Data，並取出1985及1995年的資料。
    ```r=
    CigarettesSW <- CigarettesSW %>%
        mutate(
          Rincome = income/population/cpi,
          cigtax = tax/cpi
        )
    
    df1985 <- subset(CigarettesSW, year == 1985)
    df1995 <- subset(CigarettesSW, year == 1995)
    ```
2. 生成對應的變數。
    ```r=
    # diff of packs
    pack_diff <- log(df1995$packs) - log(df1985$packs)
    
    # diff of real price
    Rprice_diff <- log(df1995$Rprice) - log(df1985$Rprice)
    
    # diff of income
    Rincome_diff <- log(df1995$Rincome) - log(df1985$Rincome)
    
    # diff of SalesTax
    SalesTax_diff <- df1995$SalesTax - df1985$SalesTax
    
    # diff of cigarettes tax
    cigtax_diff <- df1995$cigtax - df1985$cigtax
    ```
3. 3種不同的IV估計：
    （1） TSLS 只用 `SalesTax` 當 instrument。
    ```r=
    cig_ivreg_diff1 <- ivreg(pack_diff ~ Rprice_diff + Rincome_diff | Rincome_diff + SalesTax_diff)
    ```
    （2） TSLS 只用 `cigtax` 當instrument。
    ```r=
    cig_ivreg_diff2 <- ivreg(pack_diff ~ Rprice_diff + Rincome_diff | Rincome_diff + cigtax_diff)
    ```
    （3）TSLS 使用 `SalesTax` 和 `cigtax` 當instrument。
    ```r=
    cig_ivreg_diff3 <- ivreg(pack_diff ~ Rprice_diff + Rincome_diff | Rincome_diff + SalesTax_diff + cigtax_diff)
    ```
4. 3種Model比較
    ```r=
    # gather robust standard errors in a list
    rob_se <- list(sqrt(diag(vcovHC(cig_ivreg_diff1, type = "HC1"))),
              sqrt(diag(vcovHC(cig_ivreg_diff2, type = "HC1"))),
               sqrt(diag(vcovHC(cig_ivreg_diff3, type = "HC1"))))

    # generate table
    stargazer(cig_ivreg_diff1, cig_ivreg_diff2,cig_ivreg_diff3,
      header = FALSE, 
      type = "text",
      omit.table.layout = "n",
      digits = 3, 
      column.labels = c("IV: salestax", "IV: cigtax", "IVs: salestax, cigtax"),
      dep.var.labels.include = FALSE,
      dep.var.caption = "Dependent Variable: 1985-1995 Difference in Log per Pack Price",
      se = rob_se)
    ```
    
5. 上面比較的結果可以發現，`Rprice_diff`的係數估計結果差異大，因此做F檢定，來判斷第一階段回歸變數與 instrument 的相關性。
    ```r=
    # first-stage regressions
    relevance1 <- lm(Rprice_diff ~ SalesTax_diff + Rincome_diff)
    relevance2 <- lm(Rprice_diff ~ cigtax_diff + Rincome_diff)
    relevance3 <- lm(Rprice_diff ~ Rincome_diff + SalesTax_diff + cigtax_diff)
    
    # check instrument relevance for model (1)
    linearHypothesis(relevance1, 
                 "SalesTax_diff = 0", 
                 vcov = vcovHC, type = "HC1")
    
    # check instrument relevance for model (2)
    linearHypothesis(relevance2, 
                 "cigtax_diff = 0", 
                 vcov = vcovHC, type = "HC1")
    
    # check instrument relevance for model (3)
    linearHypothesis(relevance3, 
                 c("SalesTax_diff = 0", "cigtax_diff = 0"), 
                 vcov = vcovHC, type = "HC1")
    ```
    $\Rightarrow$ p-value 小，不拒絕虛無假設。
    
6. 使用J檢定，看最後殘差和 instrument 的關係。
    ```r=
    cig_iv_OR <- lm(residuals(cig_ivreg_diff3) ~ Rincome_diff + SalesTax_diff + cigtax_diff)
    
    linearHypothesis(cig_iv_OR, 
                     c("SalesTax_diff = 0", "cigtax_diff = 0"), 
                     test = "Chisq")
    # chisq 卡方檢定
    ```
    $\Rightarrow$ p-value 太大，`SalesTax` 和 `cigtax` 皆為無效 instrument。
