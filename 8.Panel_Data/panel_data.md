---
tags: R & Statistics
---
# Panel data
- 前置作業：
    ```r=
    library(dplyr)
    library(tidyverse)
    library(magrittr)
    ```
- data:
    - [變數解釋](https://bookdown.org/tpemartin/econometric_analysis/data/Fatality.docx)
    ![](https://i.imgur.com/pAg6td7.png)

    ```r=
    library(readr)
    fatality <- read_csv("https://raw.githubusercontent.com/tpemartin/Econometric-Analysis/master/Part%20II/fatality.csv")
    ```
- 檢查data:
    ```r=
    fatality %>% summarise_all(funs(class))
    ```

## Panel套件：plm
```r=
library(plm)
```
- 宣告資料為Panel data frame
    ```r=
    fatality<-pdata.frame(fatality,c("state","year"))
    ```

## 初步觀察資料
- 各州啤酒稅（beertax）與車禍死亡率（mrall）
    ```r=
    library(ggplot2)
    fatality %>% 
      ggplot()+
      geom_point(aes(x=beertax,y=I(mrall*1000)))
    ```
    - 不同州使用不同顏色畫圖
    ```r=
    fatality %>% 
      ggplot()+
      geom_point(aes(x=beertax,y=I(mrall*1000), color=state))+
      guides(color = guide_legend(keywidth = .6, keyheight = .8))
    ```
    - 不同年用不同顏色畫圖
    ```r=
    fatality %>% 
      ggplot()+
      geom_point(aes(x=beertax,y=I(mrall*1000), color=year))+
      guides(color = guide_legend(keywidth = .6, keyheight = .8))
    ```
    
## 組內差異
- 去除每個州的中間點，即進行Demean
```r=
fatality %>% 
  group_by(state) %>% #依state分組進行以下程序：
  mutate(
    mrall_demean=mrall-mean(mrall),
    beertax_demean=beertax-mean(beertax)
    ) %>%
  select(mrall_demean,beertax_demean) %>%
  ungroup() -> demean_results # grouping variable會被保留
```
- 再畫一次圖
    ```r=
    demean_results %>%
        ggplot()+
        geom_point(aes(x=beertax_demean,y=mrall_demean, color=state))+
        geom_smooth(aes(x=beertax_demean,y=mrall_demean), method = "lm",se=FALSE)+
        guides(color = guide_legend(keywidth = .6, keyheight = .8))
    ```
    
## 使用dummies
```r=
fatality %>% lm(data=., mrall~factor(state)) -> results
# results$residuals 也會是demean的結果
```
- 模型估計 迴歸模型設定
    ```r=
    model<-mrall~beertax
    ```
    
### OLS Pooled
- model: $mrall_{it} = \beta_0 +\beta_1BeerTax_{it}+v_{it}$
```r=
pool1<-plm(model, data=fatality, model='pooling')
summary(pool1)
```

### Random Effect
- model: $mrall_{it} = \beta_0 +\beta_1BeerTax_{it}+v_{it}$
    - $v_{it} = \alpha_{i} + \epsilon_{it}$
    - $\alpha_i$ 是第i州的固定效果。
- 設：
    - $v_{it} \bot BeerTax_{it}$
    - $var(\alpha_i|X) = \sigma_{\alpha}^2$
    - $Var(\epsilon|X) = \sigma^2$
    - $cov(\epsilon_{it}|X) = 0$
```r=
re1<-plm(model, data=fatality, model='random')
summary(re1)
```

### Fixed Effect
- 一個固定效果: $mrall_{it} = \alpha_i + \beta_1BeerTax_{it}+\epsilon_{it}$
    ```r=
    fe1<-plm(model, data=fatality, model='within', effect='individual')
    summary(fe1)
    ```
- 兩個固定效果: $mrall_{it} = \alpha_i + \delta_{t} + \beta_1BeerTax_{it}+\epsilon_{it}$
    - $\alpha_i$ 在整個資料中固定不變。
    - $\delta_i$ 在不同時間下的固定因子。
    ```r=
    fe2<-plm(model, data=fatality, model='within', effect='twoways')
    summary(fe2)
    ```

### Model Comparison
```r=
library(stargazer)
stargazer(pool1,re1,fe1,fe2,type='text',
          column.labels = c("Pooled OLS","RE","FE-individual","FE-two-ways"))
```
![](https://i.imgur.com/ok2V7K9.png)


## Choose Model
![](https://i.imgur.com/68I3jsQ.png)
- BP Test
    ```r=
    library(lmtest)
    bptest(pool1)
    # p-value < 0.05 => significant
    ```
- Test for Random Effects vs OLS
    ```r=
    plmtest(pool1)
    # significant
    ```
- Test for Fixed Effects versus OLS
    ```r=
    # fe1 vs pool1
    pFtest(fe1, pool1)  # significant
    # fe2 vs pool2
    pFtest(fe2, pool1)  # significant
    ```
- Hausman Test: Random Effect vs Fixed Effect
  - $H_0$: $v_{it}$ 與 $BeerTax_{it}$ 無關 -> 可以用 Fixed Effect 或 Random Effect
  - $H_1$: $v_{it}$ 與 $BeerTax_{it}$ 有關 -> 用 Fixed Effect
    ```r=
    phtest(fe1,re1)  # inconsistent => use Fixed Effect
    ```
    

    
