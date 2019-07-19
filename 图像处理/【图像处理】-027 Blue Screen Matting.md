# Blue Screen Matting

&emsp;&emsp;在电视行业，广泛使用的绿幕抠像，又称为chromakey。以前对这个应用倒是很熟悉，但对原理不熟悉，这次在阅读matting论文的时候，读到了《Blue Screen Matting》一文，感觉对蓝幕抠像、绿幕抠像的原理进行了介绍，这里对这篇文章做一个笔记。

[TOC]

## 1 简介

### 1.1 Definitions

&emsp;&emsp;抠像问题，是从一个矩形的背景图像中，分离出来一个非矩形的前景图像的问题。**matte**最开始来自电影工业，表示一卷独立的胶卷，用于记录播放时想要投影出去的画面区域，感兴趣区域是透明的。放映时，将matte和颜色胶卷同步播放，光线不能透过matte胶卷的遮蔽区域。**holdout matte**中，感兴趣区域是不透明的，其他区域都是透明的。

&emsp;&emsp;**alpha channel**是holdout matte的数字形式。对于色彩图像中需要被用户看到的像素，在alpha channel中对应一个高像素值，对于不希望用户看到的像素，在alpha channel中对应一个0值。通常，使用1和0表示alpha值，在使用8bits存储时，使用255表示1,0表示0.小数的透明度值表示彩色图像中的像素部分透明。

### 1.2 Problem

&emsp;&emsp;通过组合多个像元(**elements**)生成单一的一张结果图像(**composite**)是一个很常见的想法。这里，我们将问题限定在这样一个特殊范围内：一个或多个前景目标放置在一个特定颜色的背景上。

&emsp;&emsp;抠像问题可以视为将一个复杂场景拆分成组成这个场景的多个组成部分。这里，我们讨论的问题是只给定一幅组合后的图像，从中分离出各个组成部分。对于多通道摄影中涉及的红外、紫外等手段将前景目标记录到单一的影片中的技术，其中不涉及抠像问题。

&emsp;&emsp;使用$C_f$表示前景图像上的颜色，$\alpha_f=1$，前景图全部不透明，$C_b$表示新背景图上的颜色，$\alpha_b=1$，新背景图全部不透明。那么可以使用$C$表示组合之后的颜色，其中$C=f_{1}(C_f,C_b)$,$f_1$表示组合方式。

&emsp;&emsp;假设前景图$C_f$是由前景目标组合之前的颜色$C_o$与特定的纯色背景颜色$C_k$组合而来，那么$C_f=f_{2}(C_o,C_k)$,其中$\alpha_k = 1$,表示纯色背景全部不透明，$\alpha_o$表示前景目标上各像素的透明度。

&emsp;&emsp;在《Compositing Digital Images》一文中指出，当使用$over$操作组合两幅图像时，将透明度为$\alpha_a$的图像$a$叠加到图像$b$之上的时候，可以通过下列公式计算输出的颜色：
$$
    C=C_a+(1-\alpha_a)C_b , 0 \leq \alpha_a \leq 1 \tag{1}
$$
&emsp;&emsp;那么，对于上面的$f_2$组合，选用$over$操作，
$$
    C_f=C_o+(1-\alpha_o)C_k \tag{2}
$$
&emsp;&emsp;由于将前景图$C_f$组合到新背景图$C_b$上，指的是将前景图上的前景目标组合到新背景上，因此，
$$
    C=C_o+(1-\alpha_o)C_b \tag{3}
$$

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;**The Matting Problem**
Given $C_f$ and $C_b$ at corresponding points, and $C_k$ a known backing color, and assuming $C_f=C_o+(1-\alpha_o)C_k$, determine $C_o$ which then gives composite color $C=C_o+(1-\alpha_o)C_b$ at the corresponding point, for all points that $C_f$ and $C_b$ share in common.
&emsp;&emsp;——摘自《Blue Screen Matting》,Alvy Ray Smith and James F.Blinn,SIGGRAPH 96
&emsp;&emsp;其中，$C_o$包括颜色和透明度，称为抠像问题的解。如果各个像素的$C_o$都能够求出的话，那么就能够求出所有像素下前景图与新背景图的组合之后的颜色。

## 2 Matting problem special case solution

&emsp;&emsp;对于直接从式(2)中求解$C_o$是很困难的，因此，我们先讨论一些特殊情况下的解。

### 2.1 No blue

&emsp;&emsp;这里，我们假设前景元素中没有蓝色分量，而背景元素中只含有蓝色分量，即$c_o=[R_o,G_o,0]$,$c_k=[0,0,B_k]$,那么
$$
    c_f=c_o+(1-\alpha_o)c_k=[R_o,G_o,0]+(1-\alpha_o)[0,0,B_k]=[R_o,B_o,(1-\alpha_o)B_k] \tag{4}
$$
&emsp;&emsp;而实际上，$c_f=[R_f,G_f,B_f]$,所以
$$
    B_f = (1-\alpha_o)B_k \tag{5}
$$
&emsp;&emsp;当$B_k \neq 0$时，
$$
    \alpha_o=1-\frac{B_f}{B_k} \tag{6}
$$
&emsp;&emsp;由于前景元素中没有蓝色分量，背景只含有蓝色分量，所以，组合之后的前景图像$C_f$中所有的蓝色分量都来自于背景，因此可以求解出组合前的前景元素的透明度。

### 2.2 Gray or Flesh

&emsp;&emsp;第二类特殊问题可以理解成$C_o$是灰度图，也就是说$R_o=G_o=B_o$,我们可以将条件放松到$R_o=B_o$或者$G_o=B_o$。

&emsp;&emsp;当$R_o orG_o=aB_o+b\alpha_o$，$c_k$是纯色，并且$aB_k+b \neq 0$时，抠像问题存在解$C_o$。
$$
\begin{array}{l}
c_o=[R_o,G_o,B_o] \tag{7} & & \\
c_f=[R_f,G_f,B_f] & & \\
c_k=[0,0,B_k] & & \\
\end{array}  
$$
&emsp;&emsp;因为$C_f=C_o+(1-\alpha_o)C_k$,所以
$$
\begin{cases}
c_f=c_o+(1-\alpha_o)c_k\quad  \\\\
c_f=[R_o,G_o,B_o-(1-\alpha_o)B_k]\quad \\\\
c_f=[R_o,aB_o+b\alpha_o,B_o+(1-\alpha_o)B_k]\quad \\\\
c_f=[R_f,G_f,B_f] \tag{8}
\end{cases}
$$
&emsp;&emsp;所以
$$
\begin{cases}
G_f=aB_o+b\alpha_o\quad  \\\\
B_f=B_o+(1-\alpha_o)B_k\quad  \tag{9}
\end{cases}
$$
&emsp;&emsp;所以
$$
\begin{cases}
\alpha_{o}=\frac{G_f-aB_o}{b}\quad  \\\\
B_f=B_o+(1-\frac{G_f-aB_o}{b})B_{k}\quad  \tag{10}
\end{cases}
$$
$$
    (1+\frac{aB_{k}}{b})B_{o}=B_{f}-B_{k}+\frac{B_k}{b}G_f \tag{11}
$$
&emsp;&emsp;令$B_{\Delta}=B_{f}-B_{k}$,
$$
\begin{cases}
B_{o} &=&\frac{B_{\Delta}-\frac{B_{k}}{b}G_{f}}{1+\frac{aB_{k}}{b}}\quad  \\\\
      &=&\frac{bB_{\Delta}+B_{k}G_{f}}{b+aB_{k}}\quad  \tag{12}
\end{cases}
$$
&emsp;&emsp;因为
$$B_{f}=B_{0}+(1-\alpha_{0})B_{k} \tag{13}$$
&emsp;&emsp;所以
$$
    B_{f}-\frac{bB_{\Delta}+B_{k}G_{f}}{b+aB_{k}}=(1-\alpha_{0})B_{k} \tag{14}
$$
$$
\begin{cases}
\alpha_{0} &=&\frac{B_{f}-B_{k}-\frac{bB_{\Delta+B_{k}G_{f}}}{b+aB_{k}}}{-B_{k}}\quad  \\\\
      &=&\frac{\frac{(b+aB_{k})B_{\Delta}-bB_{\Delta}-B_{k}G_{f}}{b+aB_{k}}}{-B_{k}} \\\\
      &=&\frac{\frac{aB_{k}B_{\Delta}-B_{k}G_{f}}{b+aB_{k}}}{-B_{K}}\\\\
      &=&\frac{G_{f}-aB_{\Delta}}{b+aB_{k}} \tag{15}
\end{cases}
$$
$$
    B_{o}=B_{f}-B_{k}+\alpha_{o}B_{k}=B_{\Delta}+\alpha_{o}B_{k} \tag{16}
$$
$$
    C_{0}=[R_{f},G_{f},B_{\Delta}+\alpha_{o}B_{k},\frac{G_{f}-aB_{\Delta}}{b+aB_{k}}] \tag{17}
$$

### 2.3 Triangulation

&emsp;&emsp;对于第三种特殊情况，要求对同一个前景目标，分别在不同颜色的背景上进行两次拍摄。分别使用$B_{k_{1}}$和$B_{k_{2}}$表示两次拍摄的背景，$B_{k_{1}}=cB_{k}$,$B_{k_{2}}=dB_{k}$,$0 \leq d < c \leq 1$.
$$
    c_{f_1}=c_0+(1-\alpha_o)c_{k_{1}}=[R_o,G_o,B_o+(1-\alpha_o)B_{k_1}]=[R_{f_1},G_{f_1},B_{f_1}] \tag{18}
$$
$$
    c_{f_2}=c_0+(1-\alpha_o)c_{k_{2}}=[R_o,G_o,B_o+(1-\alpha_o)B_{k_2}]=[R_{f_2},G_{f_2},B_{f_2}] \tag{19}
$$
$$
\begin{cases}
B_{f_1} &=&B_o+(1-\alpha_o)B_{k_1}  \\\\
B_{f_2} &=&B_o+(1-\alpha_o)B_{k_2}  \tag{20}
\end{cases}
$$
$$
B_{f_1} =(B_{f_2}-(1-\alpha_o)B_{k_2})+(1-\alpha_o)B_{k_1} \tag{21}
$$
$$
    \alpha_o=1-\frac{B_{f_1}-B_{f_2}}{B_{k_1}-B_{k_2}} \tag{22}
$$
$$
    B_o=\frac{B_{k_1}B_{f_2}-B_{k_2}B_{f_1}}{B_{k_1}-B_{k_2}} \tag{23}
$$
$$
C_o=[R_{f_1},G_{f_1},\frac{B_{k_1}B_{f_2}-B_{k_2}B_{f_1}}{B_{k_1}-B_{k_2}},1-\frac{B_{f_1}-B_{f_2}}{B_{k_1}-B_{k_2}} ] \tag{24}
$$

## 3 通用解

&emsp;&emsp;令$R_{\Delta}=R_f-R_k$,$G_{\Delta}=G_f-G_k$,$B_{\Delta}=B_f-B_k$,那么
$$
   H= C_{o}\begin{bmatrix}
        1&0&0&t_{1} \\\\
        0&1&0&t_{2} \\\\
        0&0&1&t_{3} \\\\
        -R_k&-G_k&-B_k&t_{4} 
    \end{bmatrix}=[R_o,G_o,B_o,\alpha_o]\begin{bmatrix}
        1&0&0&t_{1} \\\\
        0&1&0&t_{2} \\\\
        0&0&1&t_{3} \\\\
        -R_k&-G_k&-B_k&t_{4} 
    \end{bmatrix}
$$
$$
   H= C_{o}\begin{bmatrix}
        1&0&0&t_{1} \\\\
        0&1&0&t_{2} \\\\
        0&0&1&t_{3} \\\\
        -R_k&-G_k&-B_k&t_{4} 
    \end{bmatrix}=[R_o-R_k\alpha_o,G_o-G_k\alpha_o,B_o-B_k\alpha_o,T]
$$
&emsp;&emsp;其中$T=R_ot_1+G_ot_2+B_ot_3+\alpha_ot_4$。由于
$$
\begin{aligned}
    R_f&=&R_o+(1-\alpha_o)R_k \\\\
    G_f&=&G_o+(1-\alpha_o)G_k \\\\
    B_f&=&B_o+(1-\alpha_o)B_k \tag{25}
\end{aligned}
$$
$$
\begin{aligned}
    R_{\Delta}&=&R_f - R_k=R_o-\alpha_oR_k \\\\
    G_{\Delta}&=&G_f - G_k=G_o-\alpha_oG_k \\\\
    B_{\Delta}&=&B_f - B_k=B_o-\alpha_oB_k \tag{26}
\end{aligned}
$$
&emsp;&emsp;所以
$$
H=[R_{\Delta},G_{\Delta},B_{\Delta},T] \tag{27}
$$
&emsp;&emsp;令
$$
\begin{aligned}
    \overline{t}&=&[t_1,t_2,t_3,t_4] \\\\
C_k&=&[R_k,G_k,B_k,1]
\end{aligned} \tag{28}
$$
$$
\therefore 
\begin{aligned}
\alpha_ot_4&=&T-(R_{\Delta}+\alpha_oR_k)t_1-(G_{\Delta}+\alpha_oG_k)t_2-(B_{\Delta}+\alpha_oB_k)t_3 \\\\
\alpha_o(R_kt_1+G_kt_2+B_kt_3+t_4)&=&T-(R_{\Delta}t_1+G_{\Delta}t_2+B_{\Delta}t_3)
\end{aligned} \tag{29}
$$
$$
\therefore 
\begin{aligned}
\alpha_o\overline{t}C_k&=&T-(R_{\Delta}t_1+G_{\Delta}t_2+B_{\Delta}t_3) 
\end{aligned} \tag{30}
$$
$$
\therefore 
\begin{aligned}
\alpha_o&=&\frac{T-(R_{\Delta}t_1+G_{\Delta}t_2+B_{\Delta}t_3)}{\overline{t}C_k}\\\\
 &=&\frac{T-\overline{t}C_{\Delta}}{\overline{t}C_k} \\\\
 &=&\frac{T-\overline{t}(C_f-C_k)}{\overline{t}C_k} \\\\
 &=&\frac{T-\overline{t}C_{f}+\overline{t}C_{k}}{\overline{t}C_k} \\\\
 &=&1+\frac{T-\overline{t}C_{f}}{\overline{t}C_k} \\\\
 &=&1-\frac{\overline{t}C_{f}-T}{\overline{t}C_k} \\\\
\end{aligned} \tag{31}
$$

$$
\begin{aligned}
C_o&=&C_{\Delta}+\alpha_oC_k \\\\
&=&C_f-C_k+\alpha_oC_k
\end{aligned} \tag{32}
$$

## 参考文献

- (1) Alvy Ray Smith, James F.Blinn, Blue Screen Matting
- (2) Thomas Porter, Tom Duff, Compositing Digital Images
