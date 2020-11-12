---
tags: R & Statistics
---
# R的基本操作

### R Markdown
- 可使用 Markdown 語法編輯文字、寫數學程式
- 最後可以輸出成網頁、pdf、word、ppt等檔案
- 適合報告用
- e.g. 
    - code
![example code](https://i.imgur.com/vsYdfM4.png)
    - 輸出成html檔案
![html](https://i.imgur.com/C4MP7Ex.png)


### 變數設定
- R主要是用`<-`來定義變數。
- 設定公式如下：`變數名稱 <- 變數內容（值）` 或 `變數內容（值） -> 變數名稱`
:::info
:warning: 一般都是用`變數名稱 <- 變數內容（值）`來定義變數。
- 目的：讓閱讀這個程式碼的人能先知道變數名稱，再了解變數內容。
:::

#### e.g.
```r=1
# code
a <- 1  
b <- "apple"
c <- c(1:5)
d <- c("apple", "orange", 1, "cool", 3)

# print results
print(a)  # 1
print(b)  # "apple"
print(c)  # [1] 1 2 3 4 5
print(d)  # [1] "apple"  "orange"  "1"  "cool"  "3" 
```

### 函數使用
