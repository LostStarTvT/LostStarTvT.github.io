---
layout: post
title: 一种新奇的计算平方根的方法
tags: 若有所思
---


> 在看到阮一峰周报时候看到的一种新的一元二次解题方法，特此记录。

### 一种新奇的计算平方根的方法

[原文链接](<https://www.technologyreview.com/s/614775/a-new-way-to-make-quadratic-equations-easy/>)

#### 证明
对于传统的一元二次方程组来说$x^2 + Bx + C = 0$，在求根时如果能够划分成如下：  
$$x^2 + Bx + C  = (x - R)(x - S)=0$$   
则可以立即求出来两个根$x_1 = R, x_2 = S$  
根据以上说明，求根变成求解$R,S$的值：  
$$ x^2 + Bx + C  = (x - R)(x - S)=x^2-(R+S)x + RS = 0 $$   
于是可以得出：  
$$
\begin{cases}
-B = R + S  \qquad(1)\\
 \quad C = RS \qquad  \quad (2)
\end{cases}
$$
<br>
由公式(1)我们可以设:
$$
\begin{cases}
R = - \frac{B}{2} + z  \qquad(3)\\
S = - \frac{B}{2} - z \qquad (4)
\end{cases}
$$ 
<br>
然后我们将公式(3)(4)带入公式(2)得到：  
$(- \frac{B}{2} + z)(- \frac{B}{2} - z) = (- \frac{B}{2})^2 - z^2  = C \qquad(5)$   
其中$B,C$为已知的，只有$z$为未知量，即  
$$
z=\pm \sqrt{ \frac{B^2}{4}-C}
$$ 
<br>
在求出$z$以后，带入公式(3)和(4)便可以得出$R,S$得值，也就是方程组的根。  
$$
\frac{B}{2} \pm \sqrt{ \frac{B^2}{4}-C}
$$
<br>
#### 例子
使用以上方法求出$x^2 - 2x+4 =0$的两个根。  
从上面可以得出$B=-2,C=4$, 如果直接带入最后的公式即：  
$$
\frac{-2}{2} \pm \sqrt{ \frac{(-2)^2}{4}-4} = 1\pm i\sqrt{3}
$$
<br>
其实以上主要的核心就是求出$z$值，即利用公式(5)来求  
$$
(\frac{B}{2})^2 - z^2 = 1-z^2=C=4
$$
<br>
可以解得：  
$$
z^2=-3
$$


