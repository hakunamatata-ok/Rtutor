---
tags: R & Statistics
---
> 書接上回 [R的基本操作](/DRx03K7xQvOhZ84p6jtBZQ)，詳細介紹R的矩陣。

# Matrix
- 逐行填入
    ```r=
    m1 <- matrix(c(1,2,3, 11,12,13), nrow = 2, ncol = 3, byrow = TRUE,
               dimnames = list(c("row1", "row2"),
                               c("C.1", "C.2", "C.3")))
    
    print(m1)
    ```
- 逐列填入
    ```r=
    m2 <- matrix(c(1,2,3, 11,12,13), nrow = 2, ncol = 3, byrow = F,
               dimnames = list(c("row1", "row2"),
                               c("C.1", "C.2", "C.3")))
    
    print(m2)
    ```

:::info
p.s. 
- dimname`用來設定matrix的行列名稱
- row1`和`row2`代表行的名稱，`C.1`和`C.2`代表列的名稱
- `TRUE`在R可以只用`T`表示;`FALSE`一樣可以只用`F`表示。
:::

- 看矩陣的行列數
    ```r=
    dim(m1)
    ```