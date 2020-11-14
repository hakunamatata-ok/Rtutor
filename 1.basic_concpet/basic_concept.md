---
tags: R & Statistics
---
# R的基本操作
> [Rmarkdown Code](https://github.com/hakunamatata-ok/Rtutor/blob/main/beginner/1.basic_concpet/basic_concept.Rmd)


## R Markdown
- 可使用 Markdown 語法編輯文字、寫數學程式
- 最後可以輸出成網頁、pdf、word、ppt等檔案
- 適合報告用
- e.g. 
    - Rmarkdown code 形式
![example code](https://i.imgur.com/vsYdfM4.png)
    - 輸出成html檔案
![html](https://i.imgur.com/C4MP7Ex.png)


## 變數設定
- R主要是用`<-`來定義變數。
- 設定公式如下：==`變數名稱 <- 變數內容（值）`== 或 `變數內容（值） -> 變數名稱`
:::info
:warning: 一般都是用`變數名稱 <- 變數內容（值）`來定義變數。
- 目的：讓閱讀這個程式碼的人能先知道變數名稱，再了解變數內容。
:::

#### e.g.
```r=1
# code
a <- 1
b <- "apple"
c <- c(1:10)
d <- c("apple", "orange", 1, "cool", 3)

# print results
print(a)  # 1
print(b)  # "apple"
print(c)  # [1] 1 2 3 4 5
print(d)  # [1] "apple"  "orange"  "1"  "cool"  "3" 
```
- `=`也可以定義變數。
```r=12
a = 2
print(a)  # 2
```
#### 題外話
```r=
# 取出vector中的元素
c[10]  # 10
d[2]  # "orange"
```

### 變數屬性
- `class()`：用來判斷這個變數是什麼性質，e.g. 數字（numeric）、字串（string）、布林值（logical）、列表（list）...

```r=
# type
class(a)  # numeric
class(b)  # character
class(c)  # integer
class(d)  # character
```

## 基本運算
### 數學基本運算子
運算子|意義
---|---
`+`|加法
`-`|減法
`*`|乘法
`/`|除法
`%%`|餘數|
`^`|次方|

```r=
# a=2
a+2  # 4
a-2  # 0
a*4  # 8
a/10  # 0.2
a%%1  # 0
a%%3  # 2
a^3  # 8
```

#### 進階運算
function|意義
---|---
`round(num, digit = x)`|四捨五入（num：小數，digit：將num四捨五入至x位數）
`floor(num)`|無條件捨去
`ceiling(num)`|無條件進位

```r=
round(0.3452, digit = 2)  # 0.35
floor(0.545)  # 0
ceiling(0.345)  # 1
```

### 邏輯運算子
運算子|判斷
---|---
`==`|相同
`!=`|不同
`>`|大於
`<`|小於
`>=`|大於等於
`<=`|小於等於

```r=
# a = 2
a == 2  # TRUE
a != 2  # FALSE
a > 2  # FALSE
a < 2  # FALSE
a >= 2  # TRUE
a <= 2  # TRUE
```

## Help
- 幫助了解如何運用function。
- 三種方法：
    - `?(function want to know)`, e.g. `?str`
    - `help(function want to know)`, e.g. `help(str)`
    - 直接使用 RStudio 右下角「Help」搜尋function。 
    - ![help](https://i.imgur.com/RYh4BbY.png)

## 資料結構
- vector：向量
    ![](https://media.geeksforgeeks.org/wp-content/uploads/20200409134016/Vectors-in-R.jpg)
    ```r=
    c(1, 3, 5, 9)
    c(1:10)
    c("apple", "orange", 1, 3)
    ```
- list：列表
    ```r=
    l1 <- list(c(1:5))
    l2 <- list(x = c(1,2,4))
    l3 <- list(x = c(1:10), y = c(10:20))
    print(l1)
    print(l2)
    print(l3)
    ```
    - `x`和`y`在此代表子列表的名稱
    - 取出子列表
    ```r=
    l3$x
    ```
    
- data.frame：表格
    ![](https://media.geeksforgeeks.org/wp-content/uploads/20200414224825/f115.png)
    ```r=
    df <- data.frame(col1 = c(1:10), col2 = c(11:20))
    print(df)
    ```
    - `col1`和`col2`在這裡當表格欄位名稱
    - 取出欄位值
    ```r=
    df$x
    ```
- matrix：矩陣
    ```r=
    m1 <- matrix(c(1,2,3, 11,12,13), nrow = 2, ncol = 3, byrow = TRUE,
               dimnames = list(c("row1", "row2"),
                               c("C.1", "C.2", "C.3")))
    
    print(m1)
    ```
    ```r=
    m2 <- matrix(c(1,2,3, 11,12,13), nrow = 2, ncol = 3, byrow = F,
               dimnames = list(c("row1", "row2"),
                               c("C.1", "C.2", "C.3")))
    
    print(m2)
    ```
    - `dimname`用來設定matrix的行列名稱
    - `row1`和`row2`代表行的名稱，`C.1`和`C.2`代表列的名稱
    :::info
    p.s. `TRUE`在R可以只用`T`表示;`FALSE`一樣可以只用`F`表示。
    :::

### 檢視資料內容及結構
- `str()`：object 的結構及其屬性。（object：變數的值）
```r=
str(c)
str(d)
str(l1)
str(l3)
str(df)
str(m1)
```
- `summary()`：總結、概述 object。
```r=
summary(c)
summary(l3)
summary(df)
summary(m1)
```

## 條件判斷
- `if-else`：當符合某條件時，就執行符合條件的程式，不然就執行不符合條件的程式。
    ```mermaid
    graph TB
        A --> |符合條件-TRUE|B
        A --> |不符合條件-FALSE|C
    ```
    ```r=
    if(a == 2){
      print("a is 2.")
    }else{
      print("a isn't 2.")
    }
    ```
- `if-else if-else`：當符合某條件1時，就執行符合條件1的程式，如果符合條件2，就執行符合條件2的程式，不然就執行不符合兩個條件的程式。
    ```mermaid
    graph TB
        A --> |符合條件1-TRUE|B
        A --> |不符合條件1,但符合條件2-TRUE|C
        A --> |不符合條件1,也不符合條件2|D
    ```
    ```r=
    score = 90
    if(score < 60){
      print("fail.")
    }else if((score >= 60) & (score < 70)){
      print("in level C.")
    }else if((score >= 70) & (score < 80)){
      print("in leve B.")
    }else{
      print("in level A.")
    }
    ```
- `iflese` function
    ```r=
    score1 = c(55, 60, 100)
    ifelse(score1 >= 60, "pass", "fail")
    ```

## 迴圈
- `for`：在條件下，執行程式。
    ```r=
    for (i in c(1:5)) {
      print(i)
    }
    ```
- `while`：只要符合條件，就一直執行程式。
    ```r=
    x <- 0
    while(x<=5){
      print(x)
      x <- x+1
    }
    ```
- `break`：遇到特殊狀況，可以使用`break`終止迴圈。
    ```r=
    for(n in 1:10){
      if(n==5){
        break # 一執行到5，跳出迴圈，不再執行之後的迴圈
      }
      print(n)
    }
    ```
- `next`：遇到特殊狀況，直接執行下一個迴圈
    ```r=
    for(n in 1:6){
      if(n==5){
        next # 跳過5，直接執行下一個迴圈
      }
      print(n)
    }
    ```

## 練習
1. 請列出1到10中的所有奇數
    :::spoiler
    ```r=
    for(i in 1:10){
        if((i%%2) != 0){
            print(i)
        }else{
            next
        }
    }
    ```
    :::

2. 1到10中的所有奇數都乘以2並印出，只要有一值大於10則結束迴圈。
    :::spoiler
    ```r=
    for(i in 1:10){
        if((i%%2) != 0){
            i2 <- i*2
        }else{
            next
        }
        
        if(i2 <= 10){
            print(i2)
        }else{
            break
        }
    }
    ```
    :::