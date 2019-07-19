# Compositing Digital Images

&emsp;&emsp;在阅读关于matting的论文的时候，发现每一篇论文都引用了这样一个公式：
$$
    I=F\alpha+B(1-\alpha) \tag{1}
$$
&emsp;&emsp;于是我试图去寻找这个公式的源头。在1984年Computer Graphics Volume 18,Number 3中，由Thomas Porter和Tom Duff的论文《Compositing Digital Images》中，发现了类似公式。

[TOC]

## 1 Alpha Channel

&emsp;&emsp;文中，作者为了正确的表达在图形学渲染时，前景和背景的相互关系，提出了Alpha Channel 的概念。

```C++
A separte component in needed to retain the matte information, the extent of
coverage of an element at a pixel. In a full color rendering of an element, the
RGB components retain only the color. In order to place the element over an
arbitrary background, a mixing factor is required at every pixel to control the
linear interplolation of foreground and background colors. In general, there is
no way to encode this component as part of the color information. For
anti-aliasing purposes, this mixing factor needs to be of comparable resolution
to the color channels. Let us call this an alpha channel, and let us treat an
alpha of 0 to indicate no coverage, 1 to mean full coverage, this fractions
corresponding to partial coverage.
```

&emsp;&emsp;以上文字摘自参考文献（1）。从中可以看出，$alpha-channel$具有以下性质：

- (1) 大小尺寸与图像的颜色通道一致，颜色通道中的一个像素对应alpha channel 中的一个像素；
- (2) alpha channel中的像素值，表示前景和背景的混合比例；
- (3) 0表示没有覆盖，1表示完全覆盖，0~1之间的小数表示部分覆盖。

&emsp;&emsp;那么，对于四元组$(r,g,b,a)$表示这个像素，被颜色$(r/a,g/a,b/a)$所覆盖的比例为$a$。

&emsp;&emsp;注意：这与我们通常理解的RGBA的颜色组合不一致。通常理解下，RGBA中各个分量的组合值分别为$(ra,ga,ba)$.这需要考虑RGBA格式的图像存储来理解。

## 2 RGBA格式图像文件

&emsp;&emsp; 如何直接将$(r,g,b,a)$所表示的颜色$(r/a,g/a,b/a)$存储到RGBA格式的文件中，对于全背景区域，alpha值为1,对于全前景区域，这些像素的值会为0，但对于不透明度逐渐增大的过程，会出现值从0突然增大到很大的突变过程。所以，对于RGBA格式文件，RGB值存储的是常规的RGB颜色值。

## 3 组合公式

&emsp;&emsp;组合之后的颜色可以通过图像A乘以它的透明度加上图像B乘以它的透明度。令$c_A,c_B,c_O$表示图像$A,B$和组合之后的颜色，$C_A,C_B,C_O$表示各分量在经过alpha乘以之前的颜色值，那么
$$
    c_O=\alpha_O C_O \tag{2}
$$
&emsp;&emsp;$C_O$可以经过$C_A,C_B$的加权均值计算得出，所以
$$
    c_O=\alpha_O \frac{\alpha_A F_A C_A+ \alpha_B F_B C_B}{\alpha_A F_A + \alpha_B F_B} \tag{3}
$$
&emsp;&emsp;由于分母正好是$\alpha_O=\alpha_A F_A + \alpha_B F_B$,所以
$$
    c_O=\alpha_A F_A C_A+ \alpha_B F_B C_B \tag{4}
$$
$$
    c_O = \alpha_A F_A \frac{c_A}{\alpha_A}+\alpha_B F_B \frac{c_B}{\alpha_B} \tag{5}
$$
$$
    c_O = c_A F_A + c_B F_B \tag{6}
$$

&emsp;&emsp;从公式6可以看出，输出的颜色$c_O$等于图像A的颜色$c_A$与它的透明度系数$F_A$的乘积加上图像B的颜色$c_B$与它的透明度系数$F_B$的乘积。

&emsp;&emsp;当一个像素的颜色只由图像A和B决定时，那么各个影响它的部分的透明度之和应当为1，即$F_A + F_B = 1$,那么式(6)可以写成：
$$
    c_O = c_A F_A + c_B (1-F_A) \tag{7}
$$
&emsp;&emsp;令$F_A = \alpha$,用$F$表示前景颜色，$B$表示背景颜色，$I$表示最终输出图像的颜色，那么：
$$
    I = F \alpha + B(1-\alpha) \tag{8}
$$

## 4 总结

&emsp;&emsp;《Compositing Digital Images》一文主要有以下贡献：

- (1) 提出了alpha channel的概念
- (2) 推导了不同组合模式下，前景和背景的alpha channel 值的计算形式
- (3) 推导了在over模式下，图像输出与前景、背景及alpha的计算公式：$I = F \alpha + B(1-\alpha)$

## 参考文献

- (1) Thomas Porter, Tom Duff, Compositing Digital Images, Computer Graphics, 1984
