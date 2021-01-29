---
tags: R & Statistics
---
# Difference-in-Differences Estimation
- 前置作業：
    ```r=
    library(dplyr)
    library(magrittr)
    library(ggplot2)
    library(tidyr)
    ```
## Data
- 載入名為public的data。
    - 此data為1992年4月，新西澤州將最低工資調漲前後對就業的影響。STATE = 1，為新西澤州（實驗組）；STATE = 0，為鄰州賓州（控制組）。
    - 欄位：
        | 欄位名稱 | 意義 |
        | ------- |-----|
        | STATE | 州代碼（“0”為賓州，“1”為新西澤州） |
        | EMPFT | 政策前全職員工數 |
        | EMPPT | 政策前兼職員工數 |
        | EMPFT2 | 政策後全職員工數 |
        | EMPPT2 | 政策後兼職員工數 |
    ```r=
    load(url("https://github.com/tpemartin/Econometric-Analysis/blob/master/data/public.rda?raw=true"))
    ```
- 選取欄位：STATE, EMPFT, EMPPT, EMPFT2, EMPPT2
    ```r=
    public_use <- select(public, STATE, EMPFT, EMPPT, EMPFT2, EMPPT2)
    # 查看資料屬性
    str(public_use)
    ```
- 修改資料屬性：將“character”轉變成“numeric”。
    ```r=
    public_use <- public_use %>%
      mutate_if(is.character, as.numeric)
      
    str(public_use)
    ```

## 比較資料
- 看不同州，在政策實施的前後，其就業狀況有什麼差別。
- 數字比較
    ```r=
    s_public_use <- public_use %>%
      group_by(STATE) %>%
      summarise(
        mFT_before = mean(EMPFT, na.rm = T),
        mFT_after = mean(EMPFT2, na.rm = T),
        mPT_before = mean(EMPPT, na.rm = T),
        mPT_after = mean(EMPFT2, na.rm = T),
        .groups = "drop"
      ) 
      
    s_public_use
    ```
- 畫圖比較
    - 先調整表格```s_public_use```。
    ```r=
    ns_public_use <- s_public_use %>% 
      gather(type,employment,-STATE) %>%
      mutate(
        State = ifelse(STATE == 0, "PA", "NJ")
      )
    
    ns_public_use
    ```
    - 畫圖
        - 全職就業狀況比較
        ```r=
        filter(ns_public_use, type %in% c("mFT_before", "mFT_after")) %>% 
            {ggplot(., aes(x = type, y = employment, 
                color = as.factor(State), group = State)) +
              geom_point() + 
              geom_line() +
              guides(color = guide_legend(title = "State")) +
              xlim("mFT_before", "mFT_after") +
              ggtitle(label = "Mean Full Time Employment Comparison")}
        ```
        - 兼職就業狀況比較
        ```r=
        filter(ns_public_use, type %in% c("mPT_before", "mPT_after")) %>% 
          {ggplot(., aes(x = type, y = employment, 
                color = as.factor(State), group = State)) +
              geom_point() + 
              geom_line() +
              guides(color = guide_legend(title = "State")) +
              xlim("mPT_before", "mPT_after") +
              ggtitle(label = "Mean Part Time Employment Comparison")}
        ```

## Difference-in-differences
### Model 設定
$FEmp_{ist} = FEmp_{0,,-(\alpha_s,\delta_t),ist}+\alpha_s+\delta_t+\beta*MinWage_{st}$ </br>
    $\rightarrow$ $FEmp_{ist} = \alpha_s+\delta_t+\beta*MinWage_{st}+\epsilon_{ist}$ </br>
    $\Rightarrow$ 加入虛擬變數後：
    $$FEmp_{ist}=\beta_0+\alpha_sD1_s+\delta_tB1_t+\beta_1*MinWage_{st}+\epsilon_{ist}$$
    
- 不同州（s）的第i家餐廳，在政策施行前後（t）對員工雇用量的影響。
    - $\alpha_s$：代表因不同州而異的固定效果。
    - $\delta_t$：代表時間的固定效果。
    - $D1 = 1$：實驗州的虛擬變數，即NJ。
    - $B1 = 1$：政策施行後的虛擬變數。
    - $MinWage_{st} = D1_s*B1_t$：最低工資調整的影響效果。$\rightarrow$ $MinWage_{st} = 1$ 是指該筆資料為政策施行後NJ的就業資料。
  
### Data 處理
- 先將資料轉置
    ```r=
    FT_public <- public %>% 
      select(STATE,EMPFT,EMPFT2) %>%
      group_by(STATE) %>%
      gather(type,emp,-STATE)
    ```
- 再生虛擬變數
    ```r=
    FT_public <- FT_public %>%
      mutate(
        STATE1=(STATE==1),
        AFTER=(type=="EMPFT2"),
        PolicyImpact=STATE1*AFTER
      )
      
    FT_public
    ```
### 估計
```r=
result <- lm(emp ~ STATE1+AFTER+PolicyImpact, data=FT_public)
result
```
- 回歸的統計結果
```r=
summary(result)
```

## 聚類標準誤
- 當誤差項有聚類（clustering）可能時，必需要適當的調整估計式標準誤。
- 使用package```clubSandwich```.
    ```r=
    # install.packages("clubSandwich")
    library(clubSandwich)
    ```
- 將FT_public根據STATE和type來分群.
    ```r=
    FT_public %>% 
      mutate(cluster=factor(STATE):factor(type)) -> FT_public

    FT_public$cluster %>% as.factor %>% levels
    ```
- 調整標準誤
    ```r=
    coef_test(result, vcov = "CR2", cluster = FT_public$cluster)
    ```