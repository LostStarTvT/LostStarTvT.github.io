---
layout: post
title: MarkDown中添加公式
tags: 工具使用
---


> 记录一些markdown中常用的公式，和启用的方法，其和latex输入的公式代码一致

### 启用公式JavaScript

[参考链接](https://lloyar.github.io/2018/10/08/mathjax-in-jekyll.html)  

只需要在post.html界面中引入JavaScript库即可

```javascript
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
```

### 数学公式起始和结尾标志

[参考链接](http://blog.lisp4fun.com/2017/11/01/formula)   

数学公式以 `$` 开头和结尾，例如: `\$f(x) = x^2 + 1\$` 显示为:$f(x) = x^2 + 1$

如果需要独占一行的话，则以 `$$` 开头和结尾。 例如: `$$f(x) = a + bx$$` 显示为:

$$f(x) = a + bx$$

------

### 符号上标和下标

上表用 `^` 表示，下标用 `_` 表示。

例如 `\$f(x) = a_0 + a_1 * x + a_2 * x^2\$` 显示为: $f(x) = a_0 + a_1 * x + a_2 * x^2$

------

### 多行公式

用 \begin 和 \end 把公式包围起来（支持嵌套），每行 `\\`结尾，每个元素 `&` 分隔。

公式对齐写法如下:

```
$$
\begin{align}
  f(x) = a + b \\
       = c + d  \\
\end{align}
$$
```

效果如下:

$$ \begin{align}  f(x) = a + b \\ = c + d  \\ \end{align} $$

### 多值函数

使用 `cases` 块表达式，每行 `\\`结尾，每个元素 `&` 分隔。

```
$$
p(x) = 
\begin{cases}
  p, & x = 1 \\
  1 - p, & x = 0
\end{cases}
$$
```
$$ p(x) = \begin{cases}  p, & x = 1 \\  1 - p, & x = 0 \end{cases} $$


### 矩阵

使用 `\begin{matrix}`开头及`\end{matrix}`结尾，每行 `\\`结尾，每个元素 `&`分隔。


``` 
$$ \begin{matrix}
  1 & 0 & 0 \\
  0 & 1 & 0 \\
  0 & 0 & 1 \\
\end{matrix} $$  
```

效果如下： $$ \begin{matrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \\ \end{matrix} $$ 支持的括号类型包括：

- pmatrix

$$ \begin{pmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \\ \end{pmatrix} $$

- bmatrix

$$ \begin{bmatrix}
  1 & 0 & 0 \\
  0 & 1 & 0 \\
  0 & 0 & 1 \\
\end{bmatrix} $$  

- Bmatrix

$$ \begin{Bmatrix}
  1 & 0 & 0 \\
  0 & 1 & 0 \\
  0 & 0 & 1 \\
\end{Bmatrix} $$  

- vmatrix

$$ \begin{vmatrix}
  1 & 0 & 0 \\
  0 & 1 & 0 \\
  0 & 0 & 1 \\
\end{vmatrix} $$  

- Vmatrix

$$ \begin{Vmatrix}
  1 & 0 & 0 \\
  0 & 1 & 0 \\
  0 & 0 & 1 \\
\end{Vmatrix} $$  

------

- 常用符号

| 写法             |        符号        | 备注           |
| :--------------- | :----------------: | :------------- |
| \sin(x)          |    $\sin(x)$    | 正弦函数       |
| \log(x)          |    $log(x)$    | 对数函数       |
| \sum_{i=0}^n     |    $\sum_{i=0}^n$    | 累加和         |
| \prod_{i=0}^n    |     $\prod_{i=0}^n$    | 累积乘         |
| \displaystyle    |     $\displaystyle$     | 块显示         |
| \ldots           |        $\ldots$    | 底部省略号     |
| \cdots           |        $\cdots $     | 中部省略号     |
| \int_a^b         |        $\int_a^b $      | 积分符号       |
| \lim             |       $\lim $       | 极限函数       |
| \to              |      $\to   $           | 箭头           |
| \vec{a}          |    $\vec{a}$           | 矢量a          |
| 90^\circ         |      $90^\circ$      | 度数的圆圈     |
| \uparrow         |      $\uparrow $          | 上箭头         |
| \Uparrow         |       $\Uparrow$           | 双上箭头       |
| \partial y       |       $\partial y$         | 导数/偏导      |
| \infty           |     $\infty $         | 无穷           |
| \Pi              |    $\Pi $        | 累乘           |
| \sqrt{x}         |   $\sqrt{x}$           | 求平方根       |
| \overline{a+b}   | $\overline{a+b} $ | 上划线         |
| \underline{a+b}  |     $\underline{a+b}$   | 下划线         |
| \overbrace{a+b}  |    $\overbrace{a+b}$  | 上括号         |
| \underbrace{a+b} |    $\underbrace{a+b}$  | 下括号         |
| \pm{a}{b}        |      $ \pm{a}{b} $     | 正负号         |
| \mp{a}{b}        |         $\mp{a}{b}$   | 负正号         |
| \times           |          $\times $      | 乘法           |
| \cdot            |          $\cdot $     | 点乘           |
| \ast             |          $\ast$     | 星乘           |
| \div             |           $\div$    | 除法           |
| \frac{1}{5}      |          $\frac{1}{5}$    | 分数           |
| \leq             |          $ \leq    $      | 小于等于       |
| \not             |          $\not $      | 非             |
| \geq             |          $\geq$      | 大于等于       |
| \neq             |           $\neq $     | 不等于         |
| \nleq            |           $ \nleq  $     | 不小于等于     |
| \ngeq            |          $\ngeq   $      | 不大于等于     |
| \sim             |         $ \sim   $      | 相关符号       |
| \approx          |        $\approx $         | 约等于         |
| \equiv           |          $\equiv$       | 常等于/横等于  |
| \bigodot         |        $\bigodot  $       | 加运算符       |
| \bigotimes       |      $\bigotimes$        | 乘运算符       |

------

### 集合符号

| 写法         | 符号 | 备注   |
| ------------ | :--: | ------ |
| \in          |  $\in $  | 属于   |
| \notin       |  $\notin $  | 不属于 |
| \subset      |  $\subset$  | 真子集 |
| \subseteq    |  $\subseteq$  | 子集   |
| \supset      |  $\supset$  | 超集   |
| \supseteq    |  $\supseteq $  | 超集   |
| \cup         |  $\cup  $  | 并集   |
| \cap         |  $\cap  $  | 交集   |
| \mathbb{R}   |  $\mathbb{R}$  | 实数集 |
| \emptyset    |  $\emptyset $  | 空集   |

### 希腊符号

| 写法          | 符号 |
| ------------- | :--: |
| \alpha        |  $\alpha $  |
| \beta         |  $\beta $  |
| \gamma        |  $\gamma$  |
| \Gamma        |  $ \Gamma $  |
| \theta        |  $\theta$  |
| \Theta        |  $\Theta $  |
| \delta        |  $\delta  $  |
| \Delta        |  $\Delta  $  |
| \triangledown |  $\triangledown$  |
| \epsilon      |  $\epsilon$  |
| \zeta         |  $\zeta  $  |
| \eta          |  $\eta   $  |
| \kappa        |  $\kappa $  |
| \lambda       |  $\lambda $  |
| \mu           |  $\mu  $  |
| \nu           |  $ \nu  $  |
| \xi           |  $\xi $  |
| \pi           |  $\pi  $  |
| \sigma        |  $\sigma   $  |
| \tau          |  $\tau    $  |
| \upsilon      |  $ \upsilon $  |
| \phi          |  $\phi    $  |
| \omega        |  $\omega  $  |