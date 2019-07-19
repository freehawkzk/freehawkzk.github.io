# 【MFC】-004 GDI+显示OpenCV加载的Mat矩阵

[TOC]

## 1、由来

&emsp;&emsp;OpenCV3.0取消了以前版本中用于在MFC中显示图像所使用的`Cvvimage`类。虽然能够去以前版本的OpenCV中将`Cvvimage`类源代码导出到来使用，但没有试过与3.4.2是否兼容。所以尝试将Mat转换到GDI+的Bitmap进行显示。

## 2、步骤

### 2.1 GDI+启用

&emsp;&emsp;按照常规套路启用GDI+即可。可以参考(https://blog.csdn.net/freehawkzk/article/details/80829983)进行GDI+的使用。

### 2.2 OpenCV读取图像

&emsp;&emsp;使用OpenCV读取图像并保存成Mat。

```C++
m_srcImg = cv::imread(s);//读取图像
```

&emsp;&emsp;这里有一个坑，OpenCV3.4.2默认使用MBCS字符集，而在VS2013中默认使用Unicode字符集，并且VS2013默认未安装MBCS字符集的MFC支持。当使用Unicode字符集编译OpenCV3.4.2时Release版会发生编译错误，debug版编译倒是没问题。MFC MBCS字符集程序的支持需要安装vc_mbcsmfc.exe程序，在MS官方下载网站上有。


### 2.3 GDI+显示图像

&emsp;&emsp;使用GDI+显示图像，主要使用`Gdiplus::Graphics::DrawImage`接口。该接口接收一个`Gdiplus::Bitmap`对象和一些绘制区域的参数，可以完成图像的显示。

```C++
m_pSrcBitmap = new Gdiplus::Bitmap(m_srcImg.cols, m_srcImg.rows, m_srcImg.step[0], PixelFormat24bppRGB, (BYTE*)m_srcImg.data);
```

&emsp;&emsp;以上是构造Bitmap对象并将Mat矩阵的数据传递给该对象的方法。

&emsp;&emsp;构造好Bitmap对象之后，可以使用Graphics进行显示。

```C++
		Gdiplus::Rect srcRt;
		srcRt.X = 0;
		srcRt.Y = 0;
		srcRt.Width = m_pDstBitmap->GetWidth()*m_fScaleDstImg;
		srcRt.Height = m_pDstBitmap->GetHeight()*m_fScaleDstImg;
		m_pDstGraphics->DrawImage(m_pDstBitmap, srcRt);
```

&emsp;&emsp;这里使用Bitmap对象+绘制区域矩形的方式进行绘制。

## 3、坑s

### 3.1 OpenCV3.4.2 字符集问题

&emsp;&emsp;这里有一个坑，OpenCV3.4.2默认使用MBCS字符集，而在VS2013中默认使用Unicode字符集，并且VS2013默认未安装MBCS字符集的MFC支持。当使用Unicode字符集编译OpenCV3.4.2时Release版会发生编译错误，debug版编译倒是没问题。MFC MBCS字符集程序的支持需要安装vc_mbcsmfc.exe程序，在MS官方下载网站上有。

### 3.2 GDI+ Bitmap指针构造

&emsp;&emsp;在默认MFC程序中，定义了以下宏：

```C++
#ifdef _DEBUG
#define new DEBUG_NEW
#endif
```

&emsp;&emsp;该宏会导致使用new操作符生成Bitmap指针时发生编译错误。我使用的是将该宏注释掉的方式解决这个问题。

### 3.3 Bitmap要求图像宽度4对齐

&emsp;&emsp;当图像宽度不能被4整除时，创建Bitmap对象将无法成功。

```C++
cv::resize(m_dstImg, m_dstImg, cv::Size(m_dstImg.cols / 4 * 4, m_dstImg.rows));
```

通过将图像大小重置成4的倍数来解决这个问题。