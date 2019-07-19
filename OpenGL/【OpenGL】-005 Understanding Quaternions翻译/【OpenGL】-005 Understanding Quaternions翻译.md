# 【OpenGL】-005 Understanding Quaternions翻译
&emsp;&emsp;本文是对《Understanding Quaternions》(https://www.3dgep.com/understanding-quaternions/)的翻译。

[TOC]

# Understanding Quaternions
&emsp;&emsp;本文中，将会使用便于理解的方式解释四元数的概念，四元数的表现形式，四元数支持的操作，对比矩阵、欧拉角、四元数的区别以及可以使用四元数替代矩阵或欧拉角的场景。
[![P8d4Re.png](https://s1.ax1x.com/2018/07/21/P8d4Re.png)](https://imgchr.com/i/P8d4Re)

## 1、简介
&emsp;&emsp;在计算机图形学中，使用变形矩阵表示空间中的位置或朝向。单独的一个变形矩阵可以用于表示一个物体的尺度变化或裁剪。可以认为变形矩阵式一种基础空间，当一个向量或点与变形矩阵相乘时，表示将该向量或点放入到该变形矩阵所描述的空间中。

&emsp;&emsp;本文中，不会对变形矩阵进行详细讨论，可以参考上一篇文章《Matrices》(https://www.3dgep.com/3d-math-primer-for-game-programmers-matrices/)来获取关于矩阵的详细信息。

&emsp;&emsp;四元数的概念是爱尔兰数学家Sir William Rowan Hamilton在1843年10月16日在都柏林提出的。Hamilton在与妻子一起去往Royal Irish Academy(爱尔兰皇家科学院)的路上，通过Royal Canal(皇家运河)上的Brougham Bridg(布鲁厄姆桥)的时候，突然有了一个戏剧性的发现，他立即将其刻在了桥上。
$$i^2 = j^2=k^2 = ijk = -1$$
[![P8wYSe.jpg](https://s1.ax1x.com/2018/07/21/P8wYSe.jpg)](https://imgchr.com/i/P8wYSe)

## 2、复数
&emsp;&emsp;在理解四元数之前，首先需要理解四元数的由来。四元数的概念是基于复数系统的。
&emsp;&emsp; 作为常见数字集合的补充，复数系统引入了一种新的数字集合称为虚数。虚数主要用于求解一些没有解的方程，例如：$$x^2+1=0$$
&emsp;&emsp;为了求解这个表达式，$x^2 = -1$必须成立，由于无论正数或负数的平方始终是正数，所以这个表达式被认为不可能成立。
&emsp;&emsp;数学家们不能接受一个表达式没有解，所以虚数被发明出来用于解这类表达式。
&emsp;&emsp;虚数具有如下的形式：$$i^2 = -1$$
&emsp;&emsp;不要试图去真正的理解这个表达式，因为它是没有逻辑原因的。只需要接收$i$是一个平方之后等于$-1$的东西就行。
&emsp;&emsp;虚数的集合通常使用$\mathbb{I}$表示。
&emsp;&emsp;复数的集合(使用符号$\mathbb{C}$表示)是实数和虚数的和，形式如下：
$$z=a+bi\quad a,b\in\mathbb{R},i^2=-1$$
&emsp;&emsp;可以将所有实数理解为虚部$b=0$的复数，所有虚数理解为实部$a=0$的复数。
### 2.1 复数加减法
&emsp;&emsp;复数可以通过分别对实部和虚部进行加减法来执行加减法。
**加法**$$(a_1+b_1i)+(a_2+b_2i)=(a_1+a_2)+(b_1+b_2)i$$
**减法**$$(a_1+b_1i)-(a_2+b_2i)=(a_1-a_2)+(b_1-b_2)i$$
### 2.2 复数与标量乘法
&emsp;&emsp;复数与标量的乘法是指用该标量分别乘以复数的实部和虚部：$$\lambda(a+bi)=\lambda a+\lambda b$$
### 2.3 复数乘法
&emsp;&emsp;复数的乘法遵循普通代数规则。

$$\begin{cases}
z_1\ &=&(a_1+b_1i)\\
z_2\ &=&(a_2+b_2i)\\
z_1z_2\ &=&(a_1+b_1i)(a_2+b_2i)\\
&=&a_1a_2+a_1b_2i+b_1a_2i+b_1b_2i^2\\
&=&(a_1a_2-b_1b_2)+(a_1b_2+a_2b_1)i
\end{cases}$$

### 2.4 复数平方
&emsp;&emsp;复数的平方表示复数乘以它自身。
$$\begin{cases}
z &=&(a+bi)\\
z^2&=&(a+bi)(a+bi)\\
&=&(a^2-b^2)+2abi
\end{cases}$$

### 2.5 共轭复数
&emsp;&emsp;共轭复数是指具有相同的实部，虚部是原虚部的相反数的复数，用符号$z^*$或$\overline{z}$表示。
$$
\begin{cases}
z &=&(a+bi)\\
z^* &=&(a-bi)
\end{cases}
$$
&emsp;&emsp;共轭复数的乘法结果如下：
$$
\begin{cases}
z &= (a+bi)\\
z^*&=&(a-bi)\\
zz^*&=&(a+bi)(a-bi)\\
&=&a^2-abi+abi+b^2\\
&=&a^2+b^2
\end{cases}
$$

### 2.6 复数绝对值
&emsp;&emsp;可以使用共轭复数来计算复数的绝对值(范数或模)，复数的绝对值是该复数与其共轭复数的乘积的平方根，记为$|z|$:
$$\begin{cases} 
z&=&(a+bi)\\
|z|&=&\sqrt{zz^*}\\
&=&\sqrt{(a+bi)(a-bi)}\\
&=&\sqrt{a^2+b^2}
\end{cases}$$

### 2.7 复数的商
&emsp;&emsp;计算两个复数的商，可以将分子与分母分别乘以分母的共轭复数，得到计算结果。
$$\begin{cases}
z_1&=&(a_1+b_1i)\\
z_2&=&(a_2+b_2i)\\
\frac{z_1}{z_2}&=&\frac{a_1+b_1i}{a_2+b_2i}\\
&=&\frac{(a_1+b_1i)(a_2-b_2i)}{(a_2+b_2i)(a_2-b_2i)}\\
&=&\frac{a_1a_2-a_1b_2i+b_1a_2i-b_1b_2i^2}{a_2^2+b_2^2}\\
&=&\frac{a_1a_2+b_1b_2}{a_2^2+b_2^2}+\frac{b_1a_2-a_2b_1}{a_2^2+b_2^2}i
\end{cases}$$

## 3、$i$的幂
&emsp;&emsp;因为$i^2=-1$，所以$i$的其他次幂可以按如下计算：
$$\begin{cases}
i^0&=&1\\
i^1&=&i\\
i^2&=&-1\\
i^3&=&ii^2&=&-i\\
i^4&=&i^2i^2&=&1\\
i^5&=&ii^4&=&i\\
i^6&=&ii^5&=&i^2&=&-1
\end{cases}$$
&emsp;&emsp;如果继续计算，结果符合以下规律：
$$(1,i,-1,-i,1,\dots)$$
&emsp;&emsp;$i$的负次幂也满足类似的规律：
$$\begin{cases}
i^0&=&1\\
i^{-1}&=&-i\\
i^{-2}&=&-1\\
i^{-3}&=&i\\
i^{-4}&=&1\\
i^{-5}&=&-i\\
i^{-6}&=&-1
\end{cases}$$
&emsp;&emsp;在二维笛卡尔坐标平面中，通过不断逆时针旋转90°，可以得到行如$(x,y,-x,-y,x,\dots)$的序列。通过顺时针方向旋转90°，可以得到行如$(x,-y,-x,y,x,\dots)$的序列。
[![P8fC3n.png](https://s1.ax1x.com/2018/07/21/P8fC3n.png)](https://imgchr.com/i/P8fC3n)

## 4、复平面
&emsp;&emsp;通过将实部映射到水平轴，虚部映射到垂直轴，可以将复数映射到二维平面中，称之为复平面。

[![P8fEHU.png](https://s1.ax1x.com/2018/07/21/P8fEHU.png)](https://imgchr.com/i/P8fEHU)
&emsp;&emsp;观察前面所讲到的序列，可以认为一个复数乘以$i$表示将该复数在复平面中逆时针旋转90°。
&emsp;&emsp;验证如下。
&emsp;&emsp;复平面中任意一点$p$,$$p=2+i$$
&emsp;&emsp;将$p$乘以$i$得到$q$，
$$\begin{cases}
p&=&2+i\\
q&=&pi\\
&=&(2+i)i\\
&=&2i+i^2\\
&=&-1+2i
\end{cases}$$
&emsp;&emsp;将$q$乘以$i$得到$r$，
$$\begin{cases}
q&=&-1+2i\\
r&=&qi\\
&=&(-1+2i)i\\
&=&-i+2i^2\\
&=&-2-i
\end{cases}$$
&emsp;&emsp;将$r$乘以$i$得到$s$，
$$\begin{cases}
r&=&-2-i\\
s&=&ri\\
&=&(-2-i)i\\
&=&-2i-i^2\\
&=&1-2i
\end{cases}$$
&emsp;&emsp;将$s$乘以$i$得到$t$，
$$\begin{cases}
s&=&1-2i\\
t&=&si\\
&=&(1-2i)i\\
&=&i-2i^2\\
&=&2+i
\end{cases}$$
&emsp;&emsp;此时$t$精确的回到了起点$p$.将结果在复平面中画出来，可以得到如下的结果。
[![P8f7aF.png](https://s1.ax1x.com/2018/07/21/P8f7aF.png)](https://imgchr.com/i/P8f7aF)
&emsp;&emsp;通过将复数乘以$-i$，可以得到顺时针旋转的序列。

### 4.1 旋转因子
&emsp;&emsp;定义复平面中的旋转因子$q$：$$q=cos\theta+isin\theta$$
&emsp;&emsp;任意的复数乘以旋转因子$q$：
$$\begin{cases}
p&=&a+bi\\
q&=&cos\theta+isin\theta\\
pq&=&(a+bi)(cos\theta+isin\theta)\\
a^{\prime}+b^{\prime}i&=&acos\theta-bsin\theta+(asin\theta+bcos\theta)i
\end{cases}$$
&emsp;&emsp;也可以写成如下的形式：
$$\begin{bmatrix}
a^\prime & -b^\prime\\
b^\prime & a^\prime
\end{bmatrix}=\begin{bmatrix}
cos\theta & -sin\theta\\
sin\theta & cos\theta
\end{bmatrix}
\begin{bmatrix}
a & -b\\
b & a
\end{bmatrix}
$$
&emsp;&emsp;上式表示在复平面中绕原点逆时针旋转任意点。

## 5、四元数
&emsp;&emsp;在以上关于复数和复平面的基础上，可以通过在复数的虚数中添加另外两个虚部，从而将点扩充值三维空间中。
&emsp;&emsp;四元数的通用表达形式如下：
$$q=s+xi+yj+zk\ s,x,y,z\in\mathbb{R}$$
&emsp;&emsp;通过Hamilton著名的表达式：
$$i^2=j^2=k^2=ijk=-1$$
并且
$$\begin{matrix}
ij=k  & jk=i  & ki=j\\
ji=-k & kj=-i & ik = -j 
\end{matrix}$$
&emsp;&emsp;从上可以看出$i,j,j$之间的关系类似于笛卡尔坐标系下的单位向量的叉乘：
$$\begin{matrix}
{\bf x}\times{\bf y}={\bf z} & {\bf y}\times{\bf z}={\bf x} & {\bf z}\times{\bf x}={\bf y}\\
{\bf y}\times{\bf x}=-{\bf z} & {\bf z}\times{\bf y}=-{\bf x} & {\bf x}\times{\bf z}=-{\bf y}
\end{matrix}$$
&emsp;&emsp;Hamiltion注意到$i,j,k$可以用来表示笛卡尔坐标系下的单位向量${\bf i},{\bf j},{\bf k}$,${\bf i}^2={\bf j}^2={\bf k}^2=-1$.
[![P84c1s.png](https://s1.ax1x.com/2018/07/21/P84c1s.png)](https://imgchr.com/i/P84c1s)
&emsp;&emsp;上图表示了笛卡尔坐标系下的单位向量${\bf i},{\bf j},{\bf k}$的关系。

### 5.1 四元数的有序偶形式
&emsp;&emsp;可以将四元数表示成有序偶形式：$$q=[s,{\bf v}]\ s\in\mathbb{R},{\bf v}\in\mathbb{R}^3$$
&emsp;&emsp;其中${\bf v}$也可以用它的独立分量表示：
$$q=[s,x{\bf i}+y{\bf j}+z{\bf k}]\ s,x,y,z\in\mathbb{R}$$
&emsp;&emsp;用以上形式，可以更方便的对比四元数与复数之间的相似性。

### 5.2 四元数的加减法
&emsp;&emsp;四元数的加减法类似于复数的加减法。
$$\begin{matrix}
q_a &=& [s_a,\bf{a}] \\
q_b &=& [s_b,\bf{b}]\\
q_a+q_b &=& [s_a+s_b,\bf{a}+\bf{b}]\\
q_a-q_b &=& [s_a-s_b,\bf{a}-\bf{b}]
\end{matrix}$$

### 5.3 四元数乘法
&emsp;&emsp;同样可以计算四元数的乘法：
$$\begin{matrix}
q_a &=& [s_a,{\bf a}]\\
q_b &=& [s_b,{\bf b}]\\
q_aq_b &=& [s_a {\bf a}][s_b,{\bf b}]\\
&=& (s_a+x_ai+y_aj+z_ak)(s_b+x_bi,y_bj+z_bk)\\
&=& (s_as_b-x_ax_b-y_ay_b-z_az_b)\\
& & +(s_ax_b + s_bx_a + y_az_b - y_bz_a)i\\
& & +(s_ay_b + s_by_a + z_ax_b - z_bx_a)j\\
& & +(s_az_b + s_bz_a + x_ay_b - x_by_a)k
\end{matrix}$$
&emsp;&emsp;计算结果是一个新的四元数。将虚部$i,j,k$用有序偶的形式表示：
$$ i = [0,{\bf i}] \ j=[0,{\bf j}] \ k=[0,{\bf k}]$$
&emsp;&emsp;代入上式，同时认为$[{\bf 1,0}]={\bf 1}$
$$\begin{matrix}
[s_a,{\bf a}][s_b {\bf b}] &=& (s_as_b-x_ax_b-y_ay_b-z_az_b)[{\bf 1,0}]\\
& & +(s_ax_b + s_bx_a + y_az_b - y_bz_a)[{\bf 0,i}]\\
& & +(s_ay_b + s_by_a + z_ax_b - z_bx_a)[{\bf 0,j}]\\
& & +(s_az_b + s_bz_a + x_ay_b - x_by_a)[{\bf 0,k}]
\end{matrix}$$
&emsp;&emsp;将表达式扩充成一组有序偶的和的形式：
$$\begin{matrix}
[s_a,{\bf a}][s_b,{\bf b}] &=& [s_as_b-x_ax_b-y_ay_b-z_az_b,{\bf 0}]\\
& & +[{\bf 0},(s_ax_b + s_bx_a + y_az_b - y_bz_a){\bf i}]\\
& & +[{\bf 0},(s_ay_b + s_by_a + z_ax_b - z_bx_a){\bf j}]\\
& & +[{\bf 0},(s_az_b + s_bz_a + x_ay_b - x_by_a){\bf k}]
\end{matrix}$$
&emsp;&emsp;执行四元数单位向量乘法并提取同类型，可以将上式重写成如下形式：
$$\begin{matrix} 
[s_a,{\bf a}][s_b,{\bf b}] &=& [s_as_b-x_ax_b-y_ay_b-z_az_b,{\bf 0}]\\
& & +[{\bf 0},s_a(x_b{\bf i}+y_b{\bf j}+z_b{\bf k})+s_b(x_a{\bf i}+y_a{\bf j}+z_a{\bf k})\\
& & +(y_az_b-y_bz_a){\bf i}+(z_ax_b-z_bx_a){\bf j}+(x_ay_b-x_by_a){\bf k}]
\end{matrix}$$
&emsp;&emsp;上式给出了两个有序偶的和，其中第一个是一个实四元数，第二个是一个虚四元数。这两个有序偶可以组合成一个有序偶：
$$\begin{matrix} 
[s_a,{\bf a}][s_b,{\bf b}] &=& [s_as_b-x_ax_b-y_ay_b-z_az_b,\\
& & s_a(x_b{\bf i}+y_b{\bf j}+z_b{\bf k})+s_b(x_a{\bf i}+y_a{\bf j}+z_a{\bf k})\\
& & +(y_az_b-y_bz_a){\bf i}+(z_ax_b-z_bx_a){\bf j}+(x_ay_b-x_by_a){\bf k}]
\end{matrix}$$
&emsp;&emsp;代入：
$$\begin{matrix}
{\bf a} &=& x_a{\bf i}+y_a{\bf j}+z_a{\bf k}\\
{\bf b} &=& x_b{\bf i}+y_b{\bf j}+z_b{\bf k}\\
{\bf a \cdot b} &=& x_ax_b+y_ay_b+z_az_b\\
{\bf a \times b} &=& (y_az_b-y_bz_a){\bf i}+(z_ax_b-z_bx_a){\bf j}+(x_ay_b-x_by_a){\bf k}
\end{matrix}$$
&emsp;&emsp;可得：
$$[s_a,{\bf a}][s_b,{\bf b}]=[s_as_b-{\bf a \cdot b},s_a{\bf b}+s_b{\bf a}+{\bf a \times b}]$$
&emsp;&emsp;上式是四元数乘法的通用表达式。

### 5.4 实四元数
&emsp;&emsp;实四元数是带有${\bf 0}$向量的四元数。
$$q=[s,{\bf 0}]$$
&emsp;&emsp;两个实四元数的乘积是另一个实四元数。
$$\begin{matrix}
q_a&=&[s_a,{\bf 0}]\\
q_b&=&[s_b,{\bf 0}]\\
q_aq_b&=&[s_a,{\bf 0}][s_b,{\bf 0}]\\
&=& [s_as_b,{\bf 0}]
\end{matrix}$$
&emsp;&emsp;这类似于两个虚部为0的复数的乘法：
$$\begin{matrix}
z_1&=&a_1+0i\\
z_2&=&a_2+0i\\
z_1z-2&=&(a_1+0i)(a+2+0i)\\
&=&a_1a_2
\end{matrix}$$

### 5.5 四元数与标量乘法
&emsp;&emsp;四元数与标量的乘法遵守如下规则：
$$\begin{matrix}
q&=&[s,{\bf v}]\\
\lambda q&=&\lambda[s,{\bf v}]\\
&=&[\lambda s,\lambda {\bf v}]
\end{matrix}$$
&emsp;&emsp;可以通过将标量视为一个实四元数的形式来验证：
$$\begin{matrix} 
q&=&[s,{\bf v}]\\
\lambda &=& [\lambda,{\bf 0}]\\
\lambda q&=&[\lambda,{\bf 0}][s,{\bf v}]\\
&=&[\lambda s,\lambda {\bf v}]
\end{matrix}$$

### 5.6 纯四元数
&emsp;&emsp;与实四元数类似，Hamiltion将纯四元数定义为标量部分为0的四元数。
$$q=[0,{\bf v}]$$
&emsp;&emsp;或写成分量形式：
$$q=xi+yj+zk$$
&emsp;&emsp;可以执行两个纯四元数的乘法如下：
$$\begin{matrix}
q_a &=&[0,{\bf a}]\\
q_b &=&[0,{\bf b}]\\
q_aq_b &=& [0,{\bf a}][0,{\bf b}]\\
&=& [-{\bf a \cdot b},{\bf a \times b}]
\end{matrix}$$

### 5.7 四元数的加法形式
&emsp;&emsp;可以将任意四元数表示成一个实四元数与一个纯四元数的加法：
$$\begin{matrix}
q&=&[s,{\bf v}]\\
&=&[s,{\bf 0}]+[0,{\bf v}]
\end{matrix}$$

### 5.8 单位四元数
&emsp;&emsp;对任意一个矢量${\bf v}$,可以将其表示成它的标量模和它的方向：
$${\bf v}=v\hat{{\bf v}}\ where\ v=|{\bf v}|\ and\ |\hat{{\bf v}}| = 1$$
&emsp;&emsp;将纯四元数的概念与上式相结合，可得：
$$\begin{matrix}
q&=&[0,{\bf v}]\\
&=&[0,v\hat{{\bf v}}]\\
&=&v[0,\hat{{\bf v}}]
\end{matrix}$$
&emsp;&emsp;单位四元数是标量部分为0，且矢量部分是单位向量的四元数。
$$\hat{q}=[0,\hat{{\bf v}}]$$

### 5.9 四元数的二元形式
&emsp;&emsp;将单位四元数与四元数的加法形式相结合，可以得到一种类似于复数表现形式的表示四元数的方式：
$$\begin{matrix}
q&=&[s,{\bf v}]\\
&=&[s,{\bf 0}]+[0,{\bf v}]\\
&=&[s,{\bf 0}]+v[0,\hat{{\bf v}}]\\
&=& s+v\hat{q}
\end{matrix}$$
&emsp;&emsp;上式提供了一种类似于复数表示方式的方法来表示四元数：
$$\begin{matrix}
z&=&a+bi\\
q&=&s+v\hat{q}
\end{matrix}$$

5.10 共轭四元数
&emsp;&emsp;共轭四元数可以通过将矢量部分取反得到：
$$\begin{matrix}
q&=&[s,{\bf v}]\\
q^*&=&[s,-{\bf v}]
\end{matrix}$$
&emsp;&emsp;一对共轭四元数的乘积：
$$\begin{matrix}
qq^*&=&[s,{\bf v}][s,-{\bf v}]\\
&=&[s^2-{\bf v} \cdot {\bf  -v},-s{\bf v}+s{\bf v}+{\bf v} \times {\bf -v}]\\
&=&[s^2+{\bf v \cdot v},{\bf 0}]\\
&=&[s^2+v^2,{\bf 0}]
\end{matrix}$$

### 5.11 四元数的范数
&emsp;&emsp;类似于复数的范数的计算方式：
$$\begin{matrix}
|z|&=&\sqrt{a^2+b^2}\\
zz^*&=&{|z|}^2
\end{matrix}$$
&emsp;&emsp;四元数的范数或模的计算方法定义如下：
$$\begin{matrix}
q&=&[s,{\bf v}]\\
|q|&=&\sqrt{s^2+v^2}
\end{matrix}$$
&emsp;&emsp;四元数的范数可表示成：
$$qq^* = {|q|}^2$$

### 5.12 四元数归一化
&emsp;&emsp;可以使用四元数的范数来对四元数进行归一化。四元数的归一化是指四元数除以它的模$|q|$：
$$q^\prime = \frac{q}{\sqrt{s^2+v^2}}$$
&emsp;&emsp;例子：归一化以下四元数：
$$q=[1,4{\bf i}+4{\bf j}-4{\bf k}]$$
&emsp;&emsp;首先，计算该四元数的模：
$$\begin{matrix}
|q|&=&\sqrt{1^2+4^2+4^2+(-4)^2}\\
&=&\sqrt{49}\\
&=& 7
\end{matrix}$$
&emsp;&emsp;然后，将四元数除以它的模，得到归一化的四元数：
$$\begin{matrix}
q^\prime &=& \frac{q}{|q|}\\[2ex]
&=& \frac{1+4{\bf i}+4{\bf j}-4{\bf k}}{7}\\[2ex]
&=& \frac{1}{7}+\frac{4}{7}{\bf i}+\frac{4}{7}{\bf j}-\frac{4}{7}{\bf k} \\[2ex]
\end{matrix}$$

### 5.13 四元数的逆
&emsp;&emsp;四元数的逆记为$q^{-1}$。四元数的逆等于该四元数的共轭四元数除以模的平方：
$$q^{-1}=\frac{q^*}{|q|^2}$$
&emsp;&emsp;证明过程如下：
$$qq^{-1}=[1,0]=1$$
&emsp;&emsp;两边同时乘以共轭四元数：
$$q^*qq^{-1}=q^*$$
&emsp;&emsp;代入后可得：
$$|q|^2q^{-1}=q^*$$
$$q^{-1}=\frac{q^*}{|q|^2}$$
&emsp;&emsp;对于单位模长的四元数其模长为1，那么：
$$q^{-1}=q^*$$

### 5.14 四元数的点乘
&emsp;&emsp;类似于复数的点乘，四元数的点乘可以按如下规则计算：
$$q_1=[s_1,x_1{\bf i}+y_1{\bf j}+z_1{\bf k}]$$
$$q_2=[s_2,x_2{\bf i}+y_2{\bf j}+z_2{\bf k}]$$
$$q_1 \cdot q_2=s_1s_2+x_1x_2+y_1y_2+z_1z_2$$
&emsp;&emsp;可以使用四元数的点乘来求两个四元数之间的夹角：
$$cos\theta = \frac{s_1s_2+x_1x_2+y_1y_2+z_1z_2}{|q_1||q_2|}$$
&emsp;&emsp;对于单位长度的四元数，
$$cos\theta=s_1s_2+x_1x_2+y_1y_2+z_1z_2$$

## 6、旋转
&emsp;&emsp;类似于二维复平面总的旋转因子，
$$q=cos\theta+isin\theta$$
对于三维空间中的四元数旋转形式如下：
$$q=[cos\theta, sin\theta {\bf v}]$$
&emsp;&emsp;首先，扩展矢量${\bf p}$到纯四元数形式：
$$p = [0,{\bf p}]$$
&emap;&emsp;$q$是单位模长的四元数：
$$q=[s,\lambda \hat{{\bf v}}]$$
&emsp;&emsp;然后，
$$\begin{matrix}
p^\prime &=& qp\\
&=& [s,\lambda \hat{{\bf v}}][0,{\bf p}]\\
&=& [-\lambda \hat{{\bf v}}\cdot {\bf p},s{\bf p}+\lambda \hat{{\bf v}}\times {\bf p}]
\end{matrix}$$
&emsp;&emsp;结果是一个既有标量部分又有矢量部分的四元数。
&emsp;&emsp;首先讨论特殊情况，${\bf p}$垂直与$\hat{{\bf v}}$,此时二者的点乘结果为0，$-\lambda \hat{{\bf v}} \cdot {\bf p} = 0$,于是结果变成了纯四元数形式：
$$p^\prime=[0,s{\bf p}+\lambda \hat{{\bf v}}\times {\bf p}]$$
此时，使点${\bf p}$绕${\hat{{\bf v}}}$,只需将$s=cos\theta$与$\lambda=sin\theta$代入
$$p^\prime=[0,cos\theta {\bf p}+sin\theta \hat{{\bf v}}\times {\bf p}]$$
示例，将矢量${\bf p}$绕Z轴旋转45°，那么四元数$q$：
$$q=[cos\theta,sin\theta {\bf k}] = [\frac{\sqrt{2}}{2},\frac{\sqrt{2}}{2}{\bf k}]$$
选择一个符合垂直于${\bf k}$这一特殊要求的矢量${\bf p}$:
$$p=[0,2i]$$
计算$qp$的乘积：
$$\begin{matrix}
p^\prime &=& qp\\[2ex]
&=&[\frac{\sqrt{2}}{2},\frac{\sqrt{2}}{2}{\bf k}][0,2i]\\[2ex]
&=&[0,2\frac{\sqrt{2}}{2}{\bf i}+2\frac{\sqrt{2}}{2}{\bf k}\times {\bf i}]\\[2ex]
&=&[0,\sqrt{2}{\bf i}+\sqrt{2}{\bf j}]
\end{matrix}$$
这是一个表示绕${\bf k}$轴旋转45°的纯四元数。此时，结果的模如下：
$$|p^\prime|=\sqrt{{\sqrt{2}}^2+{\sqrt{2}}^2}=2$$
与我们所期望的结果一致。
[![PGSYa4.png](https://s1.ax1x.com/2018/07/21/PGSYa4.png)](https://imgchr.com/i/PGSYa4)
&emsp;&emsp;然后在讨论与${\bf p}$不正交的四元数，以与${\bf p}$成45°角的四元数为例：
$$\begin{matrix}
\hat{{\bf v}} &=& \frac{\sqrt{2}}{2}{\bf i}+\frac{\sqrt{2}}{2}{\bf k}\\[2ex]
{\bf p}&=& 2{\bf i}\\[2ex]
q&=&[cos\theta,sin\theta \hat{{\bf v}}]\\
p&=&[0,{\bf p}]
\end{matrix}$$
将矢量${\bf p}$乘以$q$:
$$\begin{matrix}
p^\prime&=&qp\\
&=&[cos\theta,sin\theta \hat{{\bf v}}][0,{\bf p}]\\
&=&[-sin\theta \hat{{\bf v}}\cdot {\bf p},cos\theta {\bf p}+sin\theta \hat{{\bf v}} \times {\bf p}]
\end{matrix}$$
将$\hat{{\bf v}},{\bf p},\theta = 45^\circ$代入：
$$\begin{matrix}
p^\prime &=& [-\frac{\sqrt{2}}{2}(\frac{\sqrt{2}}{2}{\bf i}+\frac{\sqrt{2}}{2}{\bf k})\cdot (2{\bf i}),\frac{\sqrt{2}}{2}2{\bf i}+\frac{\sqrt{2}}{2}(\frac{\sqrt{2}}{2}{\bf i}+\frac{\sqrt{2}}{2}{\bf k})\times 2{\bf i}]\\[2ex]
&=& [-1,\sqrt{2}{\bf i}+{\bf j}]
\end{matrix}$$
结果不再是一个纯四元数，并且模不再等于2。
[![PGp6XV.png](https://s1.ax1x.com/2018/07/21/PGp6XV.png)](https://imgchr.com/i/PGp6XV)

但是，以上并没有错误。Hamilton发现如果在$qp$之后乘以$q$的逆，将得到一个纯四元数并且该四元数的模也将与被旋转向量相等。
首先，计算$q^{-1}$:
$$q=[cos\theta,sin\theta(\frac{\sqrt{2}}{2}{\bf i}+\frac{\sqrt{2}}{2}{\bf k})]$$
$$q^{-1}=[cos\theta,-sin\theta(\frac{\sqrt{2}}{2}{\bf i}+\frac{\sqrt{2}}{2}{\bf k})]$$
当$\theta = 45^\circ$时：
$$\begin{matrix}
q^{-1}&=&[\frac{\sqrt{2}}{2},-\frac{\sqrt{2}}{2}(\frac{\sqrt{2}}{2}{\bf i}+\frac{\sqrt{2}}{2}{\bf k})]\\[2ex]
&=& \frac{1}{2}[\sqrt{2},-{\bf i}-{\bf k}]
\end{matrix}$$
将$qp$与$q^{-1}$代入：
$$\begin{matrix}
qp&=&[-1,\sqrt{2}{\bf i}+{\bf j}]\\
qpq^{-1}&=&[-1,\sqrt{2}{\bf i}+{\bf j}]\frac{1}{2}[\sqrt{2},-{\bf i}-{\bf k}]\\[3ex]
&=& \frac{1}{2}[-\sqrt{2}-(\sqrt{2}{\bf i}+{\bf j})\cdot (-{\bf i}-{\bf k}),{\bf i}+{\bf k}+\sqrt{2}(\sqrt{2}{\bf i}+{\bf j})-{\bf i}+\sqrt{2}{\bf j}+{\bf k}]\\[3ex]
&=&[0,{\bf i}+\sqrt{2}{\bf j}+{\bf k}]
\end{matrix}$$
结果是纯四元数，其模为
$$\begin{matrix}
|p^\prime|&=&\sqrt{1^2+{\sqrt{2}}^2+1^2}\\[3ex]
&=&\sqrt{4}\\
&=&2
\end{matrix}$$
可见，其模长与${\bf p}$一致。
[![PG95VS.png](https://s1.ax1x.com/2018/07/21/PG95VS.png)](https://imgchr.com/i/PG95VS)
结果是一个纯四元数并且模长与初始的矢量一致，但该矢量被旋转了$90^\circ$而不是45°，是预期的角度的两倍。因此，为了正确的绕轴$\hat{{\bf v}}$旋转矢量${\bf p}$角度$\theta$，必须按照一半的角度构造旋转四元数：
$$q=[\cos{\frac{1}{2}\theta},\sin{\frac{1}{2}\theta}\hat{{\bf v}}]$$
上式是旋转四元数的通用形式。

## 7、四元数插值
&emsp;&esmp;在计算机图形学中使用四元数的一个重要原因是四元数可以很好的表示空间中的旋转。
&emsp;&emsp;四元数可以很好的克服在使用欧拉角等方式表示旋转三维空间中的点时遇到的类似万向节问题等不足。
&emsp;&emsp;使用四元数，可以定义很多种表示三维空间中的旋转的方法。第一种叫做SLERP，可以很平滑的在两个四元数之间插入一个点。第二种方法是SLERP的扩展，叫做SQAD，用于在定义的路径上的一系列点中插值。
### 7.1 SLERP


