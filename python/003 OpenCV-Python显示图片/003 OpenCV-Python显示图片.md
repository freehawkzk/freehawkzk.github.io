# 【Python】-003 OpenCV-Python显示图片

&emsp;&emsp;OpenCV提供了python绑定，能够很方便的进行图像处理。

[TOC]

## 1、OpenCV-Python的安装

&emsp;&emsp;安装好Python之后，通过pip即可进行OpenCV-Python的安装。命令如下：
```Python
pip install --upgrade setuptools
pip install numpy Matplotlib
pip install opencv-python
```

## 2、使用

&emsp;&emsp;通过Python脚本调用OpenCV,打开图片，显示图片。
```Python
#导入cv模块
import cv2 as cv
#读取图像，支持 bmp、jpg、png、tiff 等常用格式
img = cv.imread("test.png")

def GetImgWidth(image):
    shape = image.shape;
    return shape[0];

def GetImgHeight(image):
    shape = image.shape;
    return shape[1];

def GetImgChannels(image):
    shape = image.shape;
    return shape[2];

sha = GetImgWidth(img);
print(GetImgWidth(img),GetImgHeight(img),GetImgChannels(img));

#将彩色图像转换成灰度图
grayImg = cv.cvtColor(img,cv.COLOR_BGR2GRAY);

cv.imshow("gray",grayImg);
#创建窗口并显示图像
cv.namedWindow("Image")
cv.imshow("Image",img)
cv.waitKey(0)
#释放窗口
cv.destroyAllWindows()
```

## 3、结果

![python-xianshi-tuxiang](https://img-blog.csdn.net/20180823165730749?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)