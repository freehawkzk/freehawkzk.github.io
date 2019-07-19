# 【图像处理】-028 A Bayesian Approach to Digital Matting

&emsp;&emsp;抠像问题最开始从电影行业引入，在模拟摄像时代就有通过对模拟信号处理的方式进行抠像的处理办法和专利。1984年Thomas Porter和Tom Duff在《Compositing Digital Images》一文中，提出了alpha通道的概念并推导了前景和背景进行over操作时的组合图像结果像素值叠加公式。之后，微软Alvy Ray Smith推导了传统的绿幕或蓝幕抠像的数学原理，并提出了一种实现方式——对同一个目标同时拍摄两种背景下的图像，从而获得前景图像的真实颜色值和alpha值。但该方法也有弊端：(1)要求对同一目标在保持光照等因素不变的情况下获取两种背景颜色下的图像，普通技术方式难以实现。(2)算法的输入图像实质上是一种特殊要求的“自然图像”。

&emsp;&emsp;为了提高抠像算法的通用性，Yung-Yu Chuang,Brain Curless,David H.Salesin,Richard Szeliski等人在《A Bayesian Approach to Digital Matting》一文中，提出了基于贝叶斯理论的抠像算法。

[TOC]

## 1 Background

### 1.1 概述

&emsp;&emsp;对于抠像问题，有matting equation:
$$
    C=\alpha F+(1-\alpha)B \tag{1}
$$
&emsp;&emsp;其中，$C$表示组合之后的图像的颜色，$F$表示组合之前的前景图像颜色，$B$表示组合的背景颜色，$\alpha$表示前景图像中对应像素的透明度，作为前景和背景颜色进行线性组合的系数。

&emsp;&emsp;对于Blue Screen Matting,其原理是拍摄前景目标在确定的背景上的图像，然后求解前景和透明度。这显然是一个欠约束问题，我们只有3个方程但是需要求解4个未知数。
$$
\begin{aligned}
    R_c&=&\red{\alpha} \red{R_f}+(1-\red{\alpha})R_b \\\\
    G_c&=&\red{\alpha} \red{G_f}+(1-\red{\alpha})G_b \\\\
    B_c&=&\red{\alpha} \red{B_f}+(1-\red{\alpha})B_b
\end{aligned} \tag{2}
$$
&emsp;&emsp;其中，$(R_c,G_c,B_c)$,$(R_b,G_b,B_b)$均为已知，$(R_f,G_f,B_f,\alpha)$四个参数为未知数。为了求解这一问题，需要添加额外的约束条件。

&emsp;&emsp;Mishima等人提出了一种解法。对于所有的背景采样点，计算这些采样点的最小包围，中心位置为$\overline{B}$,然后对所有前景采样点，同样计算其最小包围。对于给定的组合后颜色$C$，通过$\overline{B}$和$C$做射线，分别也前景包围线、背景包围线相交于$F$、$B$点，那么
$$
\alpha = \frac{BC}{BF} \tag{3}
$$
&emsp;&emsp;这种解法的问题有：

- (1)需要在RGB空间中分别计算所有前景像素和背景像素的最小包围圈，即使简化成最小包围球体，计算量也较大；
- (2)处理结果依赖初始状态下对前景和背景的划分。

![mishima蓝幕抠像](https://img-blog.csdnimg.cn/20190701095605446.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
&emsp;&emsp;图片来自参考文献[1]

&emsp;&emsp;对于另一种特殊情况，在拍摄前景目标的图像之前，先拍摄一张只包含背景的图片，然后通过对含有前景的图像与背景图的差的阈值处理，可以将透明度值设置成0或者1，在对其进行模糊处理。这种方法称为“different matting”。这种方法的局限之处在于容易产生误差以及锯齿。

### 1.2 Trimap

&emsp;&emsp;Trimap是在matting问题中常用的一种先验知识，该图大小与输入图像一致，分为确定的前景、确定的背景和不确定区域，分别用不同的值表示这些区域。常用的一种值的表示方法是：
0表示确定的背景。255表示确定的前景区域。128表示不确定区域。

### 1.3 Knockout方法

&emsp;&emsp;Knockout方法，在给定Trimap之后，通过对前景区域和背景区域的不断外推，解决所有不确定区域的像素，得到最终的结果。其工作步骤如下：

- (1)对给定的不确定区域内的一点C，计算所有前景区域的边缘像素与C的加权均值，其中，离C最近的点的权值为1，最小距离为$l$.当距离达到$2l$时权值衰减到0，计算这一加权和，得到前景$F$;
- (2)对背景区域使用同样的方法，得到背景值$B^{'}$;
- (3)计算$\overrightarrow{FB^{'}}$,以及过$B'$点的以$\overrightarrow{FB^{'}}$为法向量的平面$S$；
- (4)将$C$投影到$S$得到点$B$;
- (5)计算$\alpha$,$$\alpha=\frac{f(C)-f(B)}{f(F)-f(B)} \tag{4}$$,其中$f(\cdot)$表示向坐标轴的投影操作。

&emsp;&emsp;![在这里插入图片描述](https://img-blog.csdnimg.cn/20190701214309428.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190701214327312.png)
&emsp;&emsp;图片来自参考文献[1]

## 2 Bayesian Method

### 2.1 Maximum A Posteriori

&emsp;&emsp;最大后验估计考量的是事件$X$已经发生的情况下，哪个$\theta$发生的概率最大。
$$
    P(\theta |X)=\frac{P(\theta)P(X|\theta)}{P(X)} \tag{5}
$$

### 2.2 Bayesian framework

&emsp;&emsp;在融合方程中，已知的只有C，而F、B和α都是未知的。于是可以从条件概率的角度去考虑这个问题，根据MAP理论，即给定C时，F、B和α的联合概率应为
$$
    P(F,B,\alpha | C) = P(C|F,B,\alpha)\times P(F,B,\alpha)/P(C) \tag{6}
$$
&emsp;&emsp;将融合问题转化为求以下函数的最大值：
$$
\argmax_{F,B,\alpha}P(F,B,\alpha|C)=\argmax_{F,B,\alpha}\frac{P(C|F,B,\alpha)P(F,B,\alpha)}{P(C)} \tag{7}
$$
&emsp;&emsp;由于$F,B,\alpha$为相互独立的随机变量，因此
$$
    P(F,B,\alpha)=P(F)P(B)P(\alpha) \tag{8}
$$
$$
\therefore \argmax_{F,B,\alpha}P(F,B,\alpha|C)=\argmax_{F,B,\alpha}\frac{P(C|F,B,\alpha)P(F)P(B)P(\alpha)}{P(C)} \tag{9}
$$
&emsp;&emsp;通过对数变换$L(\cdot)$将概率的乘法变成加法，同时由于$L(C)$为常数，对于求解最大后验估计没有影响，可以忽略，
$$
\therefore \argmax_{F,B,\alpha}P(F,B,\alpha|C)=\argmax_{F,B,\alpha}L(C|F,B,\alpha)+L(F)+L(B)+L(\alpha) \tag{10}
$$
&emsp;&emsp;通过式(10)将最终融合后颜色为$C$时，对前景$F$,背景$B$,前景$\alpha$的最大后验估计问题转换为求4项之和的最大时的参数估计。

#### 2.2.1 $L(C|F,B,\alpha)$

&emsp;&emsp;取
$$
L(C|F,B,\alpha)=-\frac{||C-\alpha F-(1-\alpha)B||^2}{2\sigma_c^{2}} \tag{11}
$$
&emsp;&emsp;表示对混合后颜色的估计误差符合均值$\overline{C}=\alpha F+(1-\alpha)B$方差为$\sigma_c$的高斯分布。

#### 2.2.2 $L(F)$

&emsp;&emsp;计算加权均值颜色$\overline{F}$和加权协方差矩阵$\sum_{F}$:
$$\overline{F}=\frac{1}{W}\sum_{i \in N}w_{i}F_{i} \tag{12}$$
$$
\sum_{F}=\frac{1}{W}\sum_{i \in N}w_{i}(F_{i}-\overline{F})(F_{i}-\overline{F})^{T} \tag{13}
$$
&emsp;&emsp;其中，$w_i=\alpha_{i}^{2}g_{i}$,$g_{i}$为方差为8的空间高斯分布。
$$
    L(F)=-\frac{(F-\overline{F})^{T}\sum_{F}^{-1}(F-\overline{F})}{2} \tag{14}
$$

#### 2.2.3 $L(B)$

&emsp;&emsp;对于$B$,处理方法和$F$一致，不过计算$w_i$时用$1-\alpha$代替$\alpha$
$$\overline{B}=\frac{1}{W}\sum_{i \in N}w_{i}B_{i} \tag{15}$$
$$
\sum_{B}=\frac{1}{W}\sum_{i \in N}w_{i}(B_{i}-\overline{B})(B_{i}-\overline{B})^{T} \tag{16}
$$
&emsp;&emsp;其中，$w_i=(1-\alpha_{i})^{2}g_{i}$,$g_{i}$为方差为8的空间高斯分布。
$$
    L(B)=-\frac{(B-\overline{B})^{T}\sum_{B}^{-1}(B-\overline{B})}{2} \tag{17}
$$

#### 2.2.4 $L(\alpha)$

&emsp;&emsp;在这里，我们认为$L(\alpha)$为常数。

### 2.3 求解

&emsp;&emsp;令$H=\argmax_{F,B,\alpha}P(F,B,\alpha|C)$,那么
$$
H=\argmax_{F,B,\alpha}L(C|F,B,\alpha)+L(F)+L(B)+L(\alpha) \tag{18}
$$
&emsp;&emsp;由于$L(C|F,B,\alpha)$中含有非完全平方项，因此，在求解时分解成两步。

#### 2.3.1 求解$F$、$B$

&emsp;&emsp;分别求$H$对$F$、$B$的偏导数，然后令偏导数等于0,得到
$$
\begin{aligned}
    \frac{\partial{H}}{\partial{F}}&=&\frac{\partial}{\partial{F}}[-\frac{||C-\alpha F-(1-\alpha)B||^2}{2\sigma_c^{2}}]+\frac{\partial}{\partial{F}}[-\frac{(F-\overline{F})^{T}\sum_{F}^{-1}(F-\overline{F})}{2}] \\\\
    &=&\frac{2I\alpha(C-\alpha F-(1-\alpha)B)}{2\sigma_c^2}-\frac{2\sum_{F}^{-1}(F-\overline{F})}{2} \\\\
    &=& \frac{I\alpha((C-\alpha F-(1-\alpha)B)}{\sigma_c^2}-\sum_{F}^{-1}  (F-\overline{F})
\end{aligned} \tag{19}
$$
$$
\begin{aligned}
    \frac{\partial{H}}{\partial{B}}&=&\frac{\partial}{\partial{B}}[-\frac{||C-\alpha F-(1-\alpha)B||^2}{2\sigma_c^{2}}]+\frac{\partial}{\partial{B}}[-\frac{(B-\overline{B})^{T}\sum_{B}^{-1}(B-\overline{B})}{2}] \\\\
    &=&\frac{2I(1-\alpha)(C-\alpha F-(1-\alpha)B)}{2\sigma_c^2}-\frac{2\sum_{F}^{-1}(B-\overline{B})}{2} \\\\
    &=& \frac{I(1-\alpha)((C-\alpha F-(1-\alpha)B)}{\sigma_c^2}-\sum_{B}^{-1}  (B-\overline{B})
\end{aligned} \tag{19}
$$
&emsp;&emsp;令$ \frac{\partial{H}}{\partial{F}}=0$,$\frac{\partial{H}}{\partial{B}}=0$,可得
$$
\begin{cases}
    \frac{I\alpha((C-\alpha F-(1-\alpha)B)}{\sigma_c^2}-\sum_{F}^{-1}  (F-\overline{F})& = &0 \\\\
    \frac{I(1-\alpha)((C-\alpha F-(1-\alpha)B)}{\sigma_c^2}-\sum_{B}^{-1}  (B-\overline{B}) &=&0
\end{cases} \tag{20}
$$
$$
\begin{cases}
    (\sum_{F}^{-1}+\frac{I\alpha^{2}}{\sigma^2})F+\frac{I\alpha(1-\alpha)}{\sigma^2}B& = & \sum_{F}^{-1}\overline{F}+\frac{I\alpha C}{\sigma^2}\\\\
    \frac{I\alpha(1-\alpha)}{\sigma^2}F+(\sum_{B}^{-1}+\frac{I(1-\alpha)^{2}}{\sigma_c^2})B &=&\sum_{B}^{-1} \overline{B}+\frac{I(1-\alpha)\alpha}{\sigma_c^2}
\end{cases} \tag{21}
$$

$$
\begin{bmatrix}
    \sum_{F}^{-1}+\frac{I\alpha^{2}}{\sigma^2}&\frac{I\alpha(1-\alpha)}{\sigma^2} \\
    \frac{I\alpha(1-\alpha)}{\sigma^2}&\sum_{B}^{-1}+\frac{I(1-\alpha)^{2}}{\sigma_c^2}
\end{bmatrix}
\begin{bmatrix}
    F\\
    B
\end{bmatrix}=
\begin{bmatrix}
    \sum_{F}^{-1}\overline{F}+\frac{I\alpha C}{\sigma^2} \\
    \sum_{B}^{-1} \overline{B}+\frac{I(1-\alpha)\alpha}{\sigma_c^2}
\end{bmatrix} \tag{22}
$$
&emsp;&emsp;对于RGB三个通道，可以形成6个方程，这里，将$\alpha$视为常数，因此可以解出$F$，$B$.

#### 2.3.2 求解$\alpha$

&emsp;&emsp;求解$\alpha$时，将$F$，$B$视为常数，因此，
$$
    \alpha = \frac{(C-B)\cdot (F-B)}{||F-B||^2} \tag{23}
$$

## 3 总结

- (1) 将融合问题转化为求解最大后验估计问题；
- (2) 通过对数操作将概率连乘转化为加法操作；
- (3) 合理的设计各分量的L函数；
- (4) 通过偏导操作可以求解部分参数；
- (5) 本算法实现时有多次循环迭代过程，计算量大，实际算法执行效率较低，无法达到实时操作。

## 参考文献

- (1) Yung-Yu Chuang, Brian Curless, David H. Salesin, Richard Szeliski, A Bayesian Approach to Digital Matting
