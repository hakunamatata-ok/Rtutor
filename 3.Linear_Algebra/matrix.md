---
tags: R & Statistics
---
> 書接上回 [R的基本操作](/DRx03K7xQvOhZ84p6jtBZQ)，詳細介紹R的矩陣。

# Matrix
## 矩陣資料結構
### 二維
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
- `dimname`用來設定matrix的行列名稱
- `row1`和`row2`代表行的名稱，`C.1`和`C.2`代表列的名稱
- `TRUE`在R可以只用`T`表示;`FALSE`一樣可以只用`F`表示。
:::

- 看矩陣的行列數
    ```r=
    dim(m1)  # 2(列) 3(行)
    ```
    - 可以使用`dim()`來改變矩陣的行列數。
    ```r=
    dim(m1) <- c(3, 2)
    m1
    ```

### 三維
- 將m1矩陣改成3維矩陣：兩個向度，每個為1:3的矩陣
    ```r=
    dim(m1) <- c(1, 3, 2)
    m1
    ```
- 此時m1的屬性會變成arry（數組）。
    ```r=
    class(m1)  # arry
    ```

### 合併列/行向量成矩陣
- `rbind()`：bind by **row**，依**行**綁定。
    ```r=
    rbind(c(1,2,3),c(4,5,6),c(7,8,9),c(10,11,12))
    ```
- `cbind()`：bind by **column**，依**列**綁定。
    ```r=
    cbind(c(1,2,3,4),c(5,6,7,8),c(9,10,11,12))
    ```
    
### 對角矩陣
- 使用`diag(x, nrow, ncol)`
    - 3*3方陣單位矩陣
    ```r=
    diag(1, nrow = 3)  
    ```
    - 4*3矩陣，對角線為1～4
    ```r=
    diag(c(1:4), nrow = 4, ncol = 3)  
    ```
    - 取出m2矩陣對角線上的數值
    ```r=
    diag(m2)
    class(diag(m2))  # "numeric"
    ```
    
### 取出矩陣元素
- 取出第一行第二列的元素
    ```r=
    m2[1,2]  # 3
    ```
- 取出第一行2～3的元素
    ```r=
    m2[1,c(2:3)]
    ```
- 取出第二行
    ```r=
    m2[2,]
    class(m2[2,])  # "numeric"
    ``` 
    - 使用`m2[2,]`取出第二行會變成一個vector，屬性為“numeric”。
    - 如果要維持為矩陣，則在`m2[2,]`中加入`drop = FALSE`。
        ```r=
        m2[2, ,drop = F]
        class(m2[2, ,drop = F])  # "matrix"
        ```
- 不顯示行列的元素
    ```r=
    m2[-2, -2]  # 不呈現第二行和第二列
    ```
- 修改矩陣元素
    ```r=
    m2[1,1] <- 3
    m2
    ```
   
### 練習
1. 請印出一個3*4的矩陣叫 mt ，依列依序填入元素1、2。
    :::spoiler
    ```r=
    mt <- matrix(1:12, nrow = 3, byrow = F)
    # "byrow = F" 可以不用寫
    ```
    :::
2. 請印出對角線為c(2,3,6)的3*4矩陣叫 mt2。
    :::spoiler
    ```r=
    mt2 <- diag(c(2, 3, 6), nrow = 3, ncol = 4)
    mt2
    ```
    :::
3. 請將 mt2 座標(3,4)的元素改成`5`。
    :::spoiler
    ```r=
    mt2[3,4] <- 5
    mt2
    ```
    :::

---
## 矩陣運算
（使用mt、mt2來做下面的運算）
### 1. 單一數字與矩陣的基本運算
- `+`
```r=
mt + 1
```
- `-`
```r=
mt - 1
```
- `*`
```r=
mt * 2
```
- `/`
```r=
mt / 2
```

### 2. 矩陣與矩陣的四則運算
:::warning
:warning: 矩陣與矩陣的陣列數需相同。
:::
- `+`
```r=
mt + mt2
```
- `-`
```r=
mt - mt2
```
- `*`
```r=
mt * mt2
```
- `/`
```r=
mt2 / mt
```

### 3. 矩陣與向量做四則運算
:::warning
:warning: 向量的長度必須小於等於陣列的行列數字。
:::
```r=
# +
mt2 + c(1:3)
mt2 + c(1:4)
# -
mt2 - c(1:3)
# *
mt2 * c(1:3)
# /
mt2/c(1:3)
```
- 
```r=
# 向量長度<矩陣行數
# +
mt2 + c(1:2)
```
- 
```r=
# 向量長度>矩陣行數 => 不適當，會出現警告訊息
mt2 + c(1:5)
```
### 4. 內積
- mt和mt2內積
```r=
# 先修改mt2矩陣行列改成4*3
dim(mt2) <- c(4,3)
mt2
```
```r=
# 做mt和mt2的內積
mt%*%mt2
```
- mt2和mt的內積
```r=
mt2%*%mt
```

### 5. 外積
- mt和m2的外積
```r=
# solution 1
mt %o% m2
```
```r=
# solution 2
outer(mt, m2)
```

### 6. 轉置矩陣
- `t()`
    ```r=
    t(m2)
    ```
    - `t()`轉置矩陣，行名稱、列名稱也會跟著改變
    ```r=
    t(m1)
    ```
### 7. 矩陣元素和平均值
function|意義
---|---
`rowSums()`|計算列總和
`colSums()`|計算行總和
`rowMeans()`|計算列平均
`colMeans()`|計算行平均
`sum()`|計算矩陣所有元素的總和
`mean()`|計算矩陣所有元素的平均

```r=
rowSums(mt2)  # 11  0  0  5
colSums(mt2)  # 2  3 11
rowMeans(mt2)  # 3.666667 0.000000 0.000000 1.666667
colMeans(mt2)  # 0.50 0.75 2.75
sum(mt2)  # 16
mean(mt2)  # 1.333333
```

### 8. 行列式值
- `det()`
    :::warning
    :warning: 只有行列數一樣的方陣可以，不同行列數的矩陣不行使用該function。
    :::
    - 方陣
    ```r=
    matrix(1:9, nrow = 3)
    det(matrix(1:9, nrow = 3))  # 0 = (1*5+5*9) - (3*5+5*7)
    ```
    - 不同行列數的矩陣
    ```r=
    det(mt2)  # Error
    ```
    
### 9. 逆矩陣
- n階方陣A和同樣為n階方陣B，其內積為n階單位方陣。
    - $AB = BA = I_n \Rightarrow B = A^{-1}$
    - $A^{-1}=\left [\begin{matrix}a&b\\ c&d \end{matrix} \right ]^{-1} =\frac{1}{ad-bc}\left [\begin{matrix}d&-b\\ -c&a \end{matrix} \right ]$
    :::warning
    :warning: 逆矩陣一定為方陣！
    :::
    ```r=
    solve(matrix(1:9, nrow = 3, ncol = 3))  # 不可逆
    ```
    ```r=
    mn <- matrix(c(1,2,5,4,9,8,7,4,9), nrow = 3, ncol = 3)
    solve(mn)  # 可逆
    ```
    - 驗證
    ```r=
     round(solve(mn) %*% mn)  # 3階單位矩陣
    ```
- 廣義逆矩陣
    - 如果$A^g$滿足$AA^gA=A$，那麼$A^g$就是廣義逆矩陣
    ```r=
    # install.packages("MASS")
    library(MASS)
    ginv(matrix(1:9, nrow = 3, ncol = 3))  # 可得出逆矩陣
    ginv(mn)  # 結果與solve(mn)相同
    ```
    - 驗證
    ```r=
    mn %*% ginv(mn) %*% mn  # 與mn相同
    ```
    ```r=
    m3 <- matrix(1:9, nrow = 3, ncol = 3)
    m3 %*% ginv(m3) %*% m3  # 與m3相同
    ```
- 解方程式
    - e.g.1 $x+3y=4, 2x+4y=6$
    ```r=
    # 未知數係數矩陣
    A <- matrix(c(1,3, 2,4), nrow = 2, byrow=T)
    # 解方程式
    solve(A) %*% matrix(c(4,6), nrow = 2)
    ```
    - e.g.2 $x+2y-z=10, 2x+3y+3z=1, 3x+4y+5z=4$
    ```r=
    A <- matrix(c(1,2,-1, 2,3,3, 3,4,5), nrow = 3, byrow = T)
    # 解方程式
    solve(A)%*%matrix(c(10,1,4), nrow = 3)
    ```
    
### 10. 特徵值和特徵向量
- 特徵值：一個常數 $\lambda$ 使(方陣A*非0向量x)和x差的倍數。
- 特徵向量：上述的非0向量x即是特徵向量。
- $Ax=\lambda x \rightarrow Ax-\lambda Ix = 0 \rightarrow (A-\lambda I)x = 0$
    - $det(\lambda I-A)=0$
    ![](https://i.imgur.com/4eEoOla.png)

    :::warning
    :warning: 只有奇異矩陣有特徵值和特徵向量。
    :::
    ```r=
    eigen(A)  # values-特徵值, vectors-特徵向量
    ```

### 11. 矩陣分解
- LU分解
    ```r=
    # install.packages("matrixcalc")
    library(matrixcalc)
    lu.decomposition(A)
    ```
    - 驗證
    ```r=
    lu.decomposition(A)$L %*% lu.decomposition(A)$U == A
    ```
- Choleskey分解
    ```r=
    m4 <- diag(4)+2
    chol(m4)
    ```
    - $分解逆矩陣*分解矩陣=原矩陣$
    ```r=
    t(chol(m4)) %*% chol(m4)
    ```
    - $分解矩陣對角線值的平方積 = 行列式值$
    ```r=
    prod(diag(chol(m4))^2) == det(m4)
    ```
- QR分解
    ```r=
    qr(m4)
    ```
    - QR分解矩陣R值
    ```r=
    qr.R(qr(m4))
    ```
    - QR分解矩陣Q值
    ```r=
    qr.Q(qr(m4))
    ```
    - $Q*R=原矩陣$
    ```r=
    qr.Q(qr(m4)) %*% qr.R(qr(m4))
    ```
    - X = 原矩陣
    ```r=
    qr.X(qr(m4))
    ```
- 奇異值分解SVD
    ```r=
    svd(m4)
    ```
    - $svd的u %*% svd的d為對角線的矩陣 %*% svd的v之轉置矩陣 = 原矩陣$
    ```r=
    svd(m4)$u %*% diag(svd(m4)$d) %*% t(svd(m4)$v)
    ```
    
### 練習
1. 請利用矩陣的方式，解未知數x,y：
    $$3x-2y = 4 \\ x-y=-2$$
    :::spoiler
    ```r=
    ma1 <- cbind(c(3,1),c(-2,-1))
    s1 <- matrix(c(4,-2), nrow = 2)
    solve(ma1) %*% s1
    # x = 8, y = 10
    ```
    :::
2. 請利用矩陣的方式，試求未知數x,y,z：
    $$3x+2y+z = 3 \\ 2x+y+z=0 \\ 6x+2y+4z=6$$
    :::spoiler
    ```r=
    ma2 <- cbind(c(3,2,6),c(2,1,2),c(1,1,4))
    s2 <- matrix(c(3,0,6), nrow = 3)
    solve(ma2) %*% s2
    # 無解
    ```
    :::
3. 請利用矩陣的方式，試求未知數x,y,z：
    $$x+3y-2z = -7 \\ 4x+y+3z=5 \\ 2x-5y+7z=19$$
    :::spoiler
    ```r=
    ma3 <- cbind(c(1,4,2),c(3,1,-5),c(-2,3,7))
    s3 <- matrix(c(-7,5,19), nrow = 3)
    solve(ma3) %*% s3
    # 無限多組解
    ```
    :::
4. 請判斷此向量集是否為線性獨立：$S = \left\{ \left[
 \begin{matrix}
   1 \\
   1 \\
   1 \\
   1 
  \end{matrix}
\right],
 \left[
  \begin{matrix}
   1 \\
   2 \\
   1 \\
   2 \\
  \end{matrix} 
 \right],
 \left[
  \begin{matrix}
  1 \\
  2 \\
  3 \\
  4 \\
  \end{matrix}
 \right]\right\}$

    :::spoiler
    ```r=
    ma4 <- cbind(c(1,1,1,1),c(1,2,1,2),c(1,2,3,4))
    t.ma4 <- t(ma4)
    det(t.ma4 %*% ma4) == 0
    # FALSE
    ```
    :::