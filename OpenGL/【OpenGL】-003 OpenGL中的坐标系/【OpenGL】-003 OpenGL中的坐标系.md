# 【OpenGL】-003 OpenGL中的几个坐标系统
&emsp;&emsp;

[TOC]

## 1、简介
&emsp;&emsp;OpenGL中，在一个物体从定义顶点坐标开始，到其被呈现到屏幕上，需要用到多个坐标系统，通常包括Model space, World space, View space, Clip space, Normalized device coordinate space, Window space等。
### 1.1 Model space
&emsp;&emsp;Model space又被称为物体坐标系，指相对物体本身原点的坐标系统，原点坐标在物体本身上，又称之为局部坐标系。例如，一个球体，它的模型坐标系或物体坐标系的原点可以在它的球心位置。

### 1.2 World space
&emsp;&emsp;World space又称为世界坐标系，用于表示物体在真实的三维空间中的坐标，世界坐标系的原点位置可以在空间中的任意位置，又称之为全局坐标系原点。通过将模型坐标系的原点经过旋转、平移等操作，可以将物体放到三维空间中的任意位置。

### 1.3 View space
&emsp;&emsp;View space又称为camera或eye，指相对观察者的位置关系。View space是指不考虑任何变换的时候目标相对观察者的关系，可以认为是一种绝对坐标系，提供了一种相对的参考。

### 1.4 Clip space
&emsp;&emsp;Clip space是OpenGL执行裁剪的坐标系。传递给Vertex shader的gl_Position坐标被认为是在裁剪坐标系中的坐标。通常是一个四位齐次坐标。

### 1.5 Normalized Device space
&emsp;&emsp;顶点坐标经过裁剪坐标系表示为一个四维齐次坐标，通过将各个分量分别除以$w$分量，即可得到顶点的NDC坐标。

### 1.6 Window space
&emsp;&emsp;顶点的像素坐标，通常是相对窗口的原点。