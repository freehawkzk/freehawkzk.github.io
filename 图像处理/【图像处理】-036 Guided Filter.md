# 【图像处理】 -036 Guided Image Filter

[toc]

## 1 引出

&emsp;&emsp;对于输入图像$p$，由于其含有噪声$n$，需要将噪声滤除。朴素的想法是通过低通滤波器，例如boxfilter或高斯滤波等对输入图像进行滤波，得到输出图像$q$,所以：
$$
q_{i} = p_{i}-n_{i} \tag{1}
$$
&emsp;&emsp;滤波器的输出等于输入减去噪声。由于对于低通滤波，通常采用的是在一个中心为$w$的窗内各个像素的加权平均。例如，对于均值滤波，图像$q$的第$i$个像素由输入图像$p$中的以第$i$个像素为中心的一个窗口$w$的所有像素的平均而求得，
$$
q_{i}=\sum_{j \in w_{i}}W_{ij}·p_{j} \tag{2}
$$
&emsp;&emsp;其中，$W_{i,j}=\frac{1}{n}$,$n$是窗口$w$中的像素个数。在窗口内部各个像素的权值相同。而高斯滤波中，$W_{ij}$符合二维高斯分布，结果导致离像素$i$近的像素将具有更高的权重，反之离像素$i$较远的像素则具有更小的权重。

&emsp;&emsp;无论是简单平滑，还是高斯平滑，它们都有一个共同的弱点，即它们都属于各向同性滤波。我们都知道，一幅自然的图像可以被看成是有（过渡平缓的，也就是梯度较小）区域和（过渡尖锐的，也就是梯度较大）边缘（也包括图像的纹理、细节等）共同组成的。噪声是影响图像质量的不利因素，我们希望将其滤除。噪声的特点通常是以其为中心的各个方向上梯度都较大而且相差不多。边缘则不同，边缘相比于区域也会出现梯度的越变，但是边缘只有在其法向方向上才会出现较大的梯度，而在切向方向上梯度较小。

&emsp;&emsp;因此，对于各向同性滤波（例如简单平滑或高斯平滑）而言，它们对待噪声和边缘信息都采取一直的态度。结果，噪声被磨平的同时，图像中具有重要地位的边缘、纹理和细节也同时被抹平了。

## 2 原理

&emsp;&emsp;导向滤波框架中，除了输入图像$p$之外，输出图像$q$之外，还需要一幅导引图$I$。导向滤波的输出被表示为一个窗口内的加权：
$$
q_{i}=\sum_{i}W_{ij}(I)p_{j} \tag{3}
$$
&emsp;&emsp;$W_{ij}$为加权权值，是导引图$I$的函数，与输入图像$p$无关。$ij$为像素坐标。

&emsp;&emsp;对于双边滤波器，其权值计算公式如下：
$$
W_{ij}^{bf}(I)=\frac{1}{K_{i}}\exp(-\frac{|\bold{x}_i-\bold{x}_j|^2}{\sigma_{s}^2})\exp(-\frac{|\bold{I}_i-\bold{I}_j|^2}{\sigma_{r}^2}) \tag{4}
$$
&emsp;&emsp;可以看出，双边滤波器的权值是空域和颜色域内的两个高斯滤波核相乘之后的结果。所以对于边缘区域，由于颜色差异较大，导致颜色域高斯滤波器输出很小，$W_{ij}$被抑制，滤波器对边缘的平滑作用被抑制，很好的保留了边缘。

&emsp;&emsp;导向滤波的一个**关键假设**是：在局部区域内，输出$q$与导引图$I$之间满足线性关系，即
$$
q_{i}=a_{k}I_{i}+b_{k}, \forall i \in w_{k} \tag{5}
$$
&emsp;&emsp;其中，$(a_k,b_k)$是线性关系的系数，在局部区域$w_k$内视为常数，$w_k$视为半径为$r$的正方形区域。这个局部线性模型，保证了只有当导引图$I$中有边缘区域时，输出图像$q$中才会有边缘，因为$\nabla q=a\nabla I$.

&emsp;&emsp;导引图像与$q$之间存在线性关系，这样设定是因为我们希望导引图像提供的是信息主要用于指示哪些是边缘哪些是区域，所以在滤波时，如果导引图告诉我们这里是区域，那么就将其磨平。如果导引图告诉我们这里是边缘，这在最终的滤波结果里就要设法保留这些边缘信息。
&emsp;&emsp;这里借用了一幅左飞老大的图。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190805161443308.png)

&emsp;&emsp;现在已知输入$p$和导引图$I$,要求输出图像$q$，如果能够求得I与q之间的线性关系$q=aI+b$,那么可以通过这个线性关系求出输出q。由于$q_{i} = p_{i}-n_{i}$,所以，可以构造目标函数为$\min||n||$,即优化使得噪声最小，
$$
\min||n|| \iff \min||n||^2 = \min\sum_{i \in w_k}(q_i-p_i) \tag{6}
$$
&emsp;&emsp;于是有
$$
\argmin \sum_{i \in w_k}(a_kI_i+b_k-p_i) \tag{7}
$$
从而将问题转化为最小二乘问题。由于普通最小而成问题会引起一些麻烦，所以要适当的引入惩罚项，采用正则化的手段。作者在处理这里时所采用的方法借鉴了从普通线性回归改进到岭回归时所采用的方法（正则化、岭回归、或者最小二乘法），即求解下面这个最优化（最小化）目标所对应的参数a和b。
$$
E(a_k,b_k)=\sum_{i \in w_k}[(a_kI_i+b_k-p_i)^2+\epsilon a_k^2] \tag{8}
$$
其中的$\epsilon$是正则化项，用于防止$a_k$取值太大。
&emsp;&emsp;求解式8之后，可得：
$$
a_k=\frac{\frac{1}{|w|}\sum_{i \in w_k}I_i p_i-\mu_k \overline{p}_k}{\sigma_k^2+\epsilon} \tag{9}
$$
$$
b_k=\overline{p}_k-a_k \mu_k \tag{10}
$$
其中，$\mu _k$是导引图$I$中窗口$w_k$中的平均值，$\sigma^2$是窗口$w_k$中的方差，$|w|$是窗口$w_k$中像素的数量，$\overline{p}_k$是输入图像$p$在窗口$w_k$中的均值，
$$
\overline{p}_k=\frac{1}{|w|}\sum_{i \in W_k}p_i \tag{11}
$$
&emsp;&emsp;在使用最小二乘计算线性系数时，由于一个像素会被多个窗包含，所以，每个像素都由多个线性函数所描述，因此，如之前所说，要具体求某一点的输出值是，只需要将所有包含该点的线性函数平均即可。
$$
q_i=\frac{1}{|w|}\sum_{k:i\in w_k}(a_k I_i + b_k)=\overline{a_i}I_i+\overline{b_i} \tag{12}
$$
其中，$\overline{a_i}=\frac{1}{|w|}\sum_{k \in w_i}a_k$,$\overline{b_i}=\frac{1}{|w|}\sum_{k \in w_i}b_k$.

&emsp;&emsp;导向滤波的算法流程如下：
fmean是一个窗口半径为r的均值滤波器，对应的窗口大小为2r+1，corr为相关，var为方差，cov为协方差。

```
Algorithm Guided Filter
Input:filtering input image p, guidance image I, radius r, regularization e
Output:filtering output q
1: mean_I=fmean(I)
mean_p=fmean(p)
corr_I=fmean(I.*I)
corr_IP=fmean(I.*p)
2:var_I=corr_I-mean_I.*mean_I
cov_IP=corr_IP-mean_I.*mean_P
3:a=cov_IP./(var_I+e)
b=mean_p-a.*mean_I
4:mean_a=fmean(a)
mean_b=fmean(b)
5:q=mean_a.*I+mean_b
```

## 3 opencv中的实现

&emsp;&emsp;在opencv中又自带的guidedFilter函数，不过是在ximgproc模块中，这个模块默认是不安装的，需要自己编译contrib包来使用。
