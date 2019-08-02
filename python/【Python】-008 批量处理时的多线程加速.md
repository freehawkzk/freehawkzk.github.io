# 【Python】-008 批量处理时的多线程加速

&emsp;&emsp;最近在处理做人脸识别的工作，准备自己撸一个人脸验证用的数据库。通过python爬虫从bing上搞了一堆图片，由于图片中不一定有人脸，所以需要弄一个人脸检测的程序先把没有人脸的图片识别出来。

&emsp;&emsp;其实，我更喜欢用C++来干这活，不过最近比较忙，我C++水平也不高，开发效率太低，所以还是用python吧，糙快猛啊。

[TOC]

## 1 人脸识别

&emsp;&emsp;Python下人脸识别很简单，可以直接用dlib的Python包搞定就行。直接pip安装就行。需要注意的是，安装pip需要先安装cmake，是cmake.exe或者cmake-gui.exe那个，并不是在Python下安装cmake包，pip install cmake并不能解决安装dlib的时候报的需要安装cmake的错误。

```python
detector = dlib.get_frontal_face_detector()
def DetectFace(imgpath):
    """
        对输入imgpath图像进行人脸检测，如果检测到的人脸数目大于0，也就是说可以是1个或多个人脸，
    返回True,否则返回False.
    """
    img = cv2.imread(imgpath) # 图片读取
    if(img is None):
        return False
    gray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY);
    dets = detector(gray, 1) # dlib进行人脸检测
    return len(dets)>0 # 图片中有无人脸
```

## 2 多线程加速

&emsp;&emsp;其实，用一个线程，现在已经可以循环着干了，一张一张的处理。但我下了20W+的图片，一张张干有点太慢了，所以我开了多线程搞。

&emsp;&emsp;在我的机器上，I7-9700K，RTX2070，16GB内存的机器，开8个线程，大概是115ms一幅，当然，我这里没有对图像大小做归一化，所以这个时间数据没有多少意义，因为图片大小不是统一的。
```python
"""
    在建立明星人脸数据库的过程中，通过爬虫爬取了大量明星图片之后，需要对图片进行过滤。
本脚本会对传入的图片路径列表文件中的所有文件，进行人脸检测，对于不能检测到人脸的图片，将其
路径保存到输出文件中。方便后续的图片过滤或删除操作。
    本脚本通过Dlib的正脸检测进行人脸检测，使用了多线程技术。
"""
import dlib
import cv2
import os
import sys
import threading
import time

dir=[]
if(len(sys.argv) == 1):
    print("You dont input the folder path that you want to list file name, dir will be default to ./!\n")
    dir="./"
else:
    dir=sys.argv[1]

savefilename=sys.argv[2]

# 创建dlib人脸检测器对象
detector = dlib.get_frontal_face_detector()
filenames=[]

def DetectFace(imgpath):
    """
        对输入imgpath图像进行人脸检测，如果检测到的人脸数目大于0，也就是说可以是1个或多个人脸，
    返回True,否则返回False.
    """
    img = cv2.imread(imgpath) # 图片读取
    if(img is None):
        return False
    gray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY);
    dets = detector(gray, 1) # dlib进行人脸检测
    return len(dets)>0 # 图片中有无人脸

def ReadImgFileList(imglistfilepath):
    """
        读取文件列表文件的内容，按行存储为list，并返回该list
    """
    lines = []
    with open(imglistfilepath, 'r') as file_to_read:
        while True:
            line = file_to_read.readline()
            if not line:
                break
            line = line.strip('\n')
            lines.append(line)
    return lines

def GetAFileName(filenames):
    """
        从输入的文件名列表中获取第一个文件名。考虑了多线程环境下的操作，
    通过定义的threading.Lock()进行多线程同步。获取了文件名后，会将该
    文件名从list中删除。
        当传入的文件名列表为空时，返回None.
    """
    threadLock.acquire()
    if len(filenames)>0:
        fn = filenames[0]
        del filenames[0]
        threadLock.release()
        return fn
    threadLock.release()
    return None

def ThreadFunc(filenames,f):
    """
        线程函数，在这里进行人脸检测，并计时。
    """
    filename = GetAFileName(filenames)
    esptime = time.clock()-starttime
    processedimg = totalnum - len(filenames)
    if processedimg % 100 == 0:
        print(str(processedimg)  + " average time : "+str(esptime/processedimg))
    if filename != None:
        if not DetectFace(filename) :
            #os.remove(file)
            #print("remove "+filename)
            f.write(filename)
            f.write("\n")
            f.flush()


class facedetectthread(threading.Thread):
    """
        人脸检测线程类，对threading.Thread的继承，完成对人脸检测函数的调用
    """
    def __init__(self, filenames, f):
        threading.Thread.__init__(self)
        self.filenames = filenames
        self.f = f
    def run(self):
        """
            线程需要执行的功能
        """
        ThreadFunc(filenames,f)

dir = dir.replace("\\\\",'/')
dir = dir.replace("\\",'/')
f = open(savefilename, 'a+')# 打开输出文件
filenames=ReadImgFileList(dir)# 读取所有文件名
threadLock = threading.Lock()# 创建获取文件名时的线程同步对象
threads = []# 记录当前已经创建的线程
starttime = time.clock()# 计时起点
totalnum = len(filenames)# 总的图像数目

while len(filenames) > 0 :# 当还有文件未处理时
    if len(threads) < 8:# 当前创建的线程总数小于8，CPU: i7-9700K
        func = facedetectthread(filenames,f)# 创建人脸检测线程对象
        func.start()# 开始执行线程
        threads.append(func)# 记录已创建的线程对象
    else:
        runfuncs=threading.enumerate()# 当前正在运行状态的线程
        for t in threads:
            if t not in runfuncs:# 对于已经结束的线程，改变其状态，并从已创建线程对象列表中去除
                t.join()
                threads.remove(t)
for t in threads:# 等待所有线程结束
    t.join()
f.close()# 关闭输出文件
```
