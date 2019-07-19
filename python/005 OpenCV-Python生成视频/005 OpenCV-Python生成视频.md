# 【Python】-005 OpenCV-Python生成视频

&emsp;&emsp;OpenCV提供了python绑定，能够很方便的进行图像处理。

[TOC]

## 1、OpenCV-Python读取视频

&emsp;&emsp;读取视频，显示，并将其中的一块区域写成新视频。

```Python
import numpy as np
import cv2 as cv
# open the video file
cap = cv.VideoCapture('d:/video/2.avi');

# get the first frame of this video
ret,frame = cap.read();

# get width and height of the video
imgHeight = frame.shape[0];
imgWidth = frame.shape[1];
print(imgWidth ,imgHeight)

# set fps and image size for the videowriter
fps = 30;
imgsize=(imgWidth,225);
fourcc = cv.VideoWriter_fourcc('M','J','P','G');

# open the video writer
videow = cv.VideoWriter('d:/video/1.avi',fourcc,fps,imgsize);

while(1):
    # take a frame from the video file
    ret ,frame = cap.read();
    # if read not success, break the loop and exit
    if ret != True:
        break;

    # show the frame read from the video
    cv.imshow('video',frame);

    # make a frame for the video writer
    newImg = frame[150:375,:];
    cv.imshow('newimg',newImg)
    print(newImg.shape[0],newImg.shape[1]);

    # write frame into file
    videow.write(newImg);

    # wait a time, if not, the window will always gray!
    cv.waitKey(60);
# close all windows opened by opencv
cv.destroyAllWindows();
# release the video file
cap.release();

# release the writer
videow.release();
```

## 2、注意

&emsp;&emsp;创建`videowriter`时的图像大小必须与最后写入的图像大小一致，否则将导致写的视频无法播放。