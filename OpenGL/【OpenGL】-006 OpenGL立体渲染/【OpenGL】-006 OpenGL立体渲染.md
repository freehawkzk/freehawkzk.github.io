# 【OpenGL】-006 OpenGL立体渲染中的缓冲区
&emsp;&emsp;最近在OpenGL立体渲染方面做了一些工作，与普通的渲染流程相比，有以下不同之处。

[TOC]

## 1、OpenGL中的颜色缓冲区

&emsp;&emsp;OpenGL中颜色缓冲区的作用如下：
- a. 颜色缓存存储了颜色索引或RGB颜色数据, 还可能存储了alpha值.
- b. 支持立体观察(stereoscopic viewing)的OpenGL实现有左颜色缓存和右颜色缓存. 它们分别用于左立体图像和右立体图像.
- c. 如不支持立体观察, 则只使用左颜色缓存.
- d. 双颜色缓存有前缓存和后缓存, 单缓存系统只有前缓存.
- e. 支持不可显示的辅助颜色缓存
- f. 函数`glGetBooleanv()`查询是否支持立体观察和双缓存: `GL_STEREO`和`GL_DOUBLE_BUFFER`.
- g. 函数`glGetIntegerv()`查询多少个辅助缓存可用: `GL_AUX_BUFFERES`. 




## 2、立体绘制时颜色缓冲区的使用

&emsp;&emsp;使用双缓存, 通常只绘制后缓存, 并在绘制完成后交换缓存. 你可能想将双缓存窗口视为单缓存窗口: 通过调用函数glDrawBuffer()使得可以同时绘制前缓存和后缓存.
```C++
void glDrawBuffer(GLenum mode)
功能: 指定要写入或消除的颜色缓存以及禁用之前被启用的颜色缓存. 可以一次性启用多个缓存.
GL_FRONT: 单缓存的默认值   
GL_FRONT_RIGHT:前右缓冲区   
GL_NONE:
GL_FRONT_LEFT:前左缓冲区
GL_FRONT_AND_BACK:
GL_RIGHT:右缓冲区
GL_AUXi: i表示第几个辅助缓存.
GL_LEFT:左缓冲区
GL_BACK_RIGHT:后右缓冲区
GL_BACK: 双缓存的默认值
GL_BACK_LEFT:后左缓冲区
注意: 启用多个缓存用于写操作时, 只要其中一个缓存存在, 就不会发生错误. 如果指定的缓存都不存在, 就发生错误.
```
## 3、立体绘制时颜色缓冲区的获取

&emsp;&emsp;在启用了立体渲染的模式下，可以通过`glReadBuffer(GLenum mode)`指定想要读取的缓冲区。
```C++
void glReadBuffer(GLenum mode);
功能: 选择接下来的函数调用glReadPixels(), glCopyPixels(), glCopyTexImage*(), glCopyTexSubImage*() 和 glCopyConvolutionFilter*()将读取的缓存.
      并启用以前被函数glReadBuffer()启用的缓存.
参数mode取值:
GL_FRONT: 单缓存默认
GL_FRONT_RIGHT:前右缓冲区
GL_BACK_RIGHT:后右缓冲区
GL_FRONT_LEFT:前左缓冲区
GL_LEFT:左缓冲区
GL_AUX:
GL_BACK_LEFT:后左缓冲区
GL_BACK: 双缓存默认
GL_RIGHT:右缓冲区
注意: 启用缓存用于读取操作时, 指定的缓存必须存在, 否则将发生错误.
```
## 4、应用

&emsp;&emsp;在Vega Prime启用立体渲染模式时，opengl开启立体模式，此时绘制所使用的是启用了`GL_BACK_LEFT`和`GL_BACK_RIGHT`的模式，在获取渲染结果时需要手动指定读取的缓冲区，否则将导致获取的结果不正确。