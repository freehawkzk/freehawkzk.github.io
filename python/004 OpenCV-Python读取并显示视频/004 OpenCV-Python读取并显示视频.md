# 【Python】-004 OpenCV-Python读取并显示视频

&emsp;&emsp;OpenCV提供了python绑定，能够很方便的进行图像处理。

[TOC]

## 1、OpenCV-Python读取视频

&emsp;&emsp;
```Python
import numpy as np
import cv2 as cv
# open the video file
cap = cv.VideoCapture('you path to the video file!');

while(1):
    # take a frame from the video file
    ret ,frame = cap.read();
    # if read not success, break the loop and exit
    if ret != True:
        break;
    # show the frame read from the video
    cv.imshow('video',frame);
    # wait a time, if not, the window will always gray!
    cv.waitKey(60);
# close all windows opened by opencv
cv.destroyAllWindows();
# release the video file
cap.release();
```

## 2、注意

&emsp;&emsp;在循环内部使用`imshow`显示图像，由于循环的作用，如果不加`waitkey`，将导致窗口显示的图像始终是灰色！