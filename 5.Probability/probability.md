---
tags: R & Statistics
---
# 機率
## 模擬隨機試驗
- 擲銅板：取後放回，共擲20次。
    ```r=
    library(caTools)
    set.seed(1234)
    f.coin <- sample(c("H","T"), size = 20, replace = T)
    f.coin
    ```
    - 看H、T各有多少個
    ```r=
    tf.coin <- table(f.coin)
    tf.coin
    ```
    - 看H、T發生佔所有事件的比例
    ```r=
    prop.table(tf.coin)
    ```
    
- 擲不公正的銅板：P(H) = 0.3, P(T) = 0.7，一樣取後放回，共擲20次。
    ```r=
    set.seed(1234)
    nf.coin <- sample(c("H","T"), size = 20, replace = T, prob = c(0.3,.07))
    nf.coin
    ```
    - 看H、T各有多少個
    ```r=
    tnf.coin <- table(nf.coin)
    tnf.coin
    ```
    - 看H、T發生佔所有事件的比例
    ```r=
    prop.table(tnf.coin)
    ```
    - 如果要強制讓實驗結果出現的比例是3:7，則使用```rep()```來設定。
        :::warning
        :warning: 注意：此時實驗就不再是隨機出現了。
        :::
    ```r=
    rep(c("H", "T"), times = c(0.3*20, 0.7*20))
    ```
    
### 練習
- 擲骰子，取後放回，共20次。（1）每個數字發生的個數；（2）每個數字發生佔所有事件的比例。（set.seed(1234))
    :::spoiler
    1. 設定模擬試驗
    ```r=
    set.seed(1234)
    dice <- sample(c(1:6), size = 20, replace = T)
    ```
    2. 每個數字發生的個數
    ```r=
    t.dice <- table(dice)
    t.dice
    ```
    3. 每個數字發生佔所有事件的比例
    ```r=
    prop.table(t.dice)
    ```
    :::
    
## 排列組合
- ```choose(n, k)```組合：從n中取出k。
    ```r=
    choose(4,2)
    ```
- ```factorial(k)```階層：k!
    ```r=
    factorial(3)
    ```
- ```combn(x,n)```：列出所有組合數的矩陣
    ```r=
    combn(4,2)
    t(combn(4,2))  # 轉置
    ```
    - ```combinations(n, k)```
    ```r=
    library(gtools)
    combinations(4,2)  # 結果與t(combn(4,2))一樣
    ```
- ```expand.grid()```：以data.frame的形式輸出所有組合數的狀況，有出現是1，沒有出現是0。
    ```r=
    # 列出2個變數，看其所有組合的狀況
    expand.grid(rep(list(0:1),2))
    ```
- ```permutations(n, k)```排列：從n個元素中，取出k個來排列。此function在```gtools``` package 內。
    ```r=
    library(gtools)
    permutations(4, 2)
    ```
    
## 條件機率
- 撲克牌data。
    - spade 黑桃
    - heart 紅心
    - diamond 方塊
    - club 梅花
```r=
card <- data.frame(color = rep(c("spade","heart","diamond","club"), 13), num = rep(c(1:13),4))

table(card)
```
- 取出第一張牌是紅心，取出第二張牌也是紅心的機率。
    ```r=
    # 第一張牌
    card1 <- choose(NROW(card[card$color == "heart",]),1)/choose(NROW(card),1)
    # 第二張牌
    card2 <- choose(NROW(card[card$color == "heart",]),2)/choose(NROW(card),2)

    card2/card1  # 0.2352941
    ```
### 練習
- 從撲克牌中取出5張牌，前4張都是黑桃（spade），請問第5張也是黑桃的機率。
    :::spoiler
    ```r=
    # 前四張抽出來是黑桃的機率
    B <- choose(NROW(card[card$color=="spade",]), 4)/choose(NROW(card), 4)
    # 第5張也是黑桃的機率
    AB <- choose(NROW(card[card$color=="spade",]), 5)/choose(NROW(card), 5)

    AB/B
    ```
    :::
    

## 應用
### Monty Hall Problem
- 3道門，其中一道後面是車子，另外兩道門後都是山羊。主持人知道每道門後面是什麼，但抽的人不知道。如果抽的人選了一道有山羊的門，那麼主持人就要選另一道一樣有山羊的門；但如果抽的人選有車子的門，那麼主持人會隨機選一道有山羊的門。當選定後，主持人會問抽的人是否要換？問：如果換一道門，抽中車子的機率會不會增加？
- code
    ```r=
    B <- 10000
    monty_hall <- function(strategy){
      doors <- as.character(1:3)
      prize <- sample(c("car", "goat", "goat"))
      prize_door <- doors[prize == "car"]
      my_pick  <- sample(doors, 1)
      show <- sample(doors[!doors %in% c(my_pick, prize_door)],1)
      stick <- my_pick
      stick == prize_door
      switch <- doors[!doors %in% c(my_pick, show)]
      choice <- ifelse(strategy == "stick", stick, switch)
      choice == prize_door
    }
    
    stick <- replicate(B, monty_hall("stick"))
    mean(stick)  # 0.3369
    
    switch <- replicate(B,monty_hall("switch"))
    mean(switch)  # 0.669
    ```
