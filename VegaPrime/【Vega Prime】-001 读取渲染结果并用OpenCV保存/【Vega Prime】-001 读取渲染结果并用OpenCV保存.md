# 【Vega Prime】-001 读取渲染结果并用OpenCV保存
&emsp;&emsp;最近遇到一个问题，在使用VP进行渲染之后，需要把多个channel中的画面读取出来，并保存成图片和视频。问题不难，权当一个笔记吧。
[TOC]
## 1、VP渲染
&emsp;&emsp;这里我对VP的渲染没做改动，从vpApp派生出我自己的app类之后，全部按照默认流程干就行。
## 2、使用OpenGL获取渲染结果
### 2.1 让VP在完成帧渲染时发出通知
&emsp;&emsp;为了能够让VP的完成帧渲染之后发送完成渲染的通知，需要为响应的通道订阅**EVENT_POST_DRAW**事件。` m_pRightEyeChannel->addSubscriber(vsChannel::EVENT_POST_DRAW, this);`通过这句来订阅事件。需要注意为了成功添加事件订阅，app类需要增加从`vsChannel::Subscriber`类的派生，并重写相对应的`notify`函数。
```C++
class CErgonomicTestViewApp: public vpApp, vsChannel::Subscriber
{
    /**
     * inherited pre / post draw notification method. Since we're only
     * subscribed to post draw events we don't need to bother checking for the
     * pre draw events.
     */
    virtual void notify(vsChannel::Event, const vsChannel *channel,
        vrDrawContext *context) ;
};
```
&emsp;&emsp;由于我们只订阅了`EVENT_POST_DRAW`事件，所以在`notify`函数中我们并不需要进行事件类型的检查。如果为单个通道订阅了多个事件，在处理之前需要判断通道并判断事件类型做出相应的处理。
&emsp;&emsp;**notify**函数的第一个参数`vsChannel::Event`是当前通知的事件，第二个参数`const vsChannel *channel`是当前通知的channel，最后一个参数`vrDrawContext *context`是VP维护的一个绘制上下文。
### 2.2 获取buffer
&emsp;&emsp;获取VP渲染buffer的主要代码如下：
```C++
    int ox, oy, sx, sy;
    pChannel->getVrChannel()->getViewport(&ox, &oy, &sx, &sy);//读取当前视口的参数
    //获取当前buffer内容，并保存到m_data中
    glReadPixels(ox, oy, sx, sy, GL_BGR, GL_UNSIGNED_BYTE, m_data);
```
这里`m_data`是用户定义的`unsigned char *m_data;`,该指针在VP执行`configure`函数是完成初始化，分配足够大小的内存空间。对于不改变Viewport大小的应用，在VP执行完vp自己的configure之后，各个通道的viewport大小此时已经是已知的，所以对于视口所对应的窗口大小已经是固定的，因此，可以在configure中为m_data申请空间。
`m_data = vuAllocArray<uchar >::malloc(MAX_WINDOW_WIDTH*MAX_WINDOW_HEIGHT*3);`,**注意**，由于m_data是用户自己申请的内存，在程序退出或异常时，需要手动释放。
```C++
    if(m_data != NULL)
        vuAllocArray<uchar >::free(m_data);
```
## 3、使用OpenCV保存渲染结果
&emsp;&emsp;此时，渲染结果已经被保存到m_data中，只需要将这个缓冲区保存下来即可。对于这个缓冲区的保存，方法很多，个人根据自己熟悉的方法做就行。
&emsp;&emsp;由于之前使用FreeImage做图像加载和保存已经完成，但在使用freeimage时，发现将图像保存成avi以及MP4文件时比较吃力，所以转而求助于OpenCV。
### 3.1 将buffer保存到FIBITMAP
&emsp;&emsp;在完成了FIBITMAP对象的创建时，需要指定图像的宽度和高度，可以直接使用viewport的大小。
```C++
    //创建FIBITMAP对象
    m_pLeftImg = FreeImage_Allocate(nWidth,nHeight,24,8,8,8);
    //获取FIBITMAP对象的数据指针
    BYTE* pData  = FreeImage_GetBits(pBitmap);
    //将图像数据拷贝从m_data缓冲区拷贝到FIBITMAP对象中
    memcpy(pData,m_data,sx*sy*3);
```
### 3.2 FIBITMAP转存到cv::Mat
&emsp;&ensp;这里准备了一个从FIBITMAP结构转换到cv::Mat结构的函数。实际上从m_data直接创建cv::Mat的方法更好，并且肯定能够实现，不过我暂时没有时间去折腾，所以就拿来主义吧。
```C++
void FI2MAT(FIBITMAP* src, cv::Mat& dst)
{
    //FIT_BITMAP    //standard image : 1 - , 4 - , 8 - , 16 - , 24 - , 32 - bit
    //FIT_UINT16    //array of unsigned short : unsigned 16 - bit
    //FIT_INT16     //array of short : signed 16 - bit
    //FIT_UINT32    //array of unsigned long : unsigned 32 - bit
    //FIT_INT32     //array of long : signed 32 - bit
    //FIT_FLOAT     //array of float : 32 - bit IEEE floating point
    //FIT_DOUBLE    //array of double : 64 - bit IEEE floating point
    //FIT_COMPLEX   //array of FICOMPLEX : 2 x 64 - bit IEEE floating point
    //FIT_RGB16     //48 - bit RGB image : 3 x 16 - bit
    //FIT_RGBA16    //64 - bit RGBA image : 4 x 16 - bit
    //FIT_RGBF      //96 - bit RGB float image : 3 x 32 - bit IEEE floating point
    //FIT_RGBAF     //128 - bit RGBA float image : 4 x 32 - bit IEEE floating point

    int bpp = FreeImage_GetBPP(src);
    FREE_IMAGE_TYPE fit = FreeImage_GetImageType(src);

    int cv_type = -1;
    int cv_cvt = -1;

    switch (fit)
    {
    case FIT_UINT16: cv_type = cv::DataType<ushort>::type; break;
    case FIT_INT16: cv_type = cv::DataType<short>::type; break;
    case FIT_UINT32: cv_type = cv::DataType<unsigned>::type; break;
    case FIT_INT32: cv_type = cv::DataType<int>::type; break;
    case FIT_FLOAT: cv_type = cv::DataType<float>::type; break;
    case FIT_DOUBLE: cv_type = cv::DataType<double>::type; break;
    case FIT_COMPLEX: cv_type =cv::DataType<cv::Complex<double>>::type; break;
    case FIT_RGB16: cv_type = cv::DataType<cv::Vec<ushort, 3>>::type; cv_cvt = cv::COLOR_RGB2BGR; break;
    case FIT_RGBA16: cv_type = cv::DataType<cv::Vec<ushort, 4>>::type; cv_cvt = cv::COLOR_RGBA2BGRA; break;
    case FIT_RGBF: cv_type = cv::DataType<cv::Vec<float, 3>>::type; cv_cvt = cv::COLOR_RGB2BGR; break;
    case FIT_RGBAF: cv_type = cv::DataType<cv::Vec<float, 4>>::type; cv_cvt = cv::COLOR_RGBA2BGRA; break;
    case FIT_BITMAP:
        switch (bpp) {
    case 8: cv_type = cv::DataType<cv::Vec<uchar, 1>>::type; break;
    case 16: cv_type = cv::DataType<cv::Vec<uchar, 2>>::type; break;
    case 24: cv_type = cv::DataType<cv::Vec<uchar, 3>>::type; break;
    case 32: cv_type = cv::DataType<cv::Vec<uchar, 4>>::type; break;
    default:
        // 1, 4 // Unsupported natively
        cv_type = -1;
        }
        break;
    default:
        // FIT_UNKNOWN // unknown type
        dst = cv::Mat(); // return empty Mat
        return;
    }

    int width = FreeImage_GetWidth(src);
    int height = FreeImage_GetHeight(src);
    int step = FreeImage_GetPitch(src);

    if (cv_type >= 0) {
        dst = cv::Mat(height, width, cv_type, FreeImage_GetBits(src), step);
        if (cv_cvt > 0)
        {
            cv::cvtColor(dst, dst, cv_cvt);
        }
    }
    else {

        std::vector<uchar> lut;
        int n = (long)pow((long double)2,(int) bpp);
        for (int i = 0; i < n; ++i)
        {
            lut.push_back(static_cast<uchar>((255 / (n - 1))*i));
        }

        FIBITMAP* palletized = FreeImage_ConvertTo8Bits(src);
        BYTE* data = FreeImage_GetBits(src);
        for (int r = 0; r < height; ++r) {
            for (int c = 0; c < width; ++c) {
                dst.at<uchar>(r, c) = cv::saturate_cast<uchar>(lut[data[r*step + c]]);
            }
        }
    }

    flip(dst, dst, 0);
}
```
### 3.3 保存图像
&emsp;&emsp;至此图像数据已经到OpenCV的cv::Mat里，之后的图像保存对OpenCV来说实在是很简单的事。通过cv::imwrite即可完成。
```C++
cv::imwrite(sFileName.c_str(),img_merge);
```
### 3.4 保存视频
&emsp;&emsp;OpenCV使用cv::VideoWriter进行视频的写入。
```C++
    if (!m_pVideoWriter)
    {
        cv::Size sz(img_merge.cols,img_merge.rows);
        std::string sAviName = sImgRootPath + sAVI+sScensName+".avi";
        m_pVideoWriter = new cv::VideoWriter(sAviName.c_str(), CV_FOURCC('M', 'J', 'P', 'G'), 10, sz);
    }
    m_pVideoWriter->write(img_merge);
```