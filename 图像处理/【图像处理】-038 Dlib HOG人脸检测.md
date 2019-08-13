# 【图像处理】 -038 Dlib Hog人脸检测

[toc]

## 1 简介

&emsp;&emsp;方向梯度直方图（英语：Histogram of oriented gradient，简称HOG）是应用在计算机视觉和图像处理领域，用于目标检测的特征描述器。这项技术是用来计算局部图像梯度的方向信息的统计值。这种方法跟边缘方向直方图（edge orientation histograms）、尺度不变特征变换（scale-invariant feature transform descriptors）以及形状上下文方法（ shape contexts）有很多相似之处，但与它们的不同点是：HOG描述器是在一个网格密集的大小统一的细胞单元（dense grid of uniformly spaced cells）上计算，而且为了提高性能，还采用了重叠的局部对比度归一化（overlapping local contrast normalization）技术。

&emsp;&emsp;HOG特征又称为方向梯度直方图特征。HOG描述器最重要的思想是：在一副图像中，局部目标的表象和形状（appearance and shape）能够被梯度或边缘的方向密度分布很好地描述。具体的实现方法是：首先将图像分成小的连通区域，我们把它叫细胞单元。然后采集细胞单元中各像素点的梯度的或边缘的方向直方图。最后把这些直方图组合起来就可以构成特征描述器。为了提高性能，我们还可以把这些局部直方图在图像的更大的范围内（我们把它叫区间或block）进行对比度归一化（contrast-normalized），所采用的方法是：先计算各直方图在这个区间（block）中的密度，然后根据这个密度对区间中的各个细胞单元做归一化。通过这个归一化后，能对光照变化和阴影获得更好的效果。

&emsp;&emsp;与其他的特征描述方法相比，HOG描述器有很多优点。首先，由于HOG方法是在图像的局部细胞单元上操作，所以它对图像几何的（geometric）和光学的（photometric）形变都能保持很好的不变性，这两种形变只会出现在更大的空间领域上。其次，作者通过实验发现，在粗的空域抽样（coarse spatial sampling）、精细的方向抽样（fine orientation sampling）以及较强的局部光学归一化（strong local photometric normalization）等条件下，只要行人大体上能够保持直立的姿势，就容许行人有一些细微的肢体动作，这些细微的动作可以被忽略而不影响检测效果。综上所述，HOG方法是特别适合于做图像中的行人检测的。

## 2 使用dlib实现HOG人脸检测

```C++
// Dlib_HOG.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//

#include <iostream>
#include <string>
#include <fstream>
#include "opencv2/opencv.hpp"
#include "../OpenCV_Harr/OpenCV_Harr/HighPerformanceTimer.hpp"
#include <dlib/opencv.h>
#include <dlib/image_processing.h>
#include <dlib/image_processing/frontal_face_detector.h>

//读取待检测文件列表
std::vector<std::string> ReadImgList(std::string& imglistfilename)
{
	std::vector<std::string> imgs;
	std::ifstream imglistfile(imglistfilename, std::ifstream::in);
	std::string line;
	while (getline(imglistfile, line))//按行读取
	{
		imgs.push_back(line);
	}
	return imgs;
}


int main(int argc,char** argv)
{
	if (argc < 3)
	{
		std::cout << "Please use this exe like this:" << std::endl;
		std::cout << "OpenCV_Harrx.exe imglist.txt outputpath" << std::endl;
		system("pause");
	}
	std::string imglistfile(argv[1]);
	std::string outputpath(argv[2]);
	
	//创建Dlib正脸检测器
	dlib::frontal_face_detector hogFaceDetector = dlib::get_frontal_face_detector();
	
	//加载待检测的图片路径列表
	std::vector<std::string> imgs = ReadImgList(imglistfile);

	//创建计时器对象
	char* pTimerName = (char*)"Dlib-hog";
	CHighPerformanceTimer* pTimer = new CHighPerformanceTimer(pTimerName, 9, true);

	std::ofstream of(outputpath, std::ofstream::out);
	//循环处理所有图片
	for (int i = 0; i < imgs.size(); i++)
	{
		//读取图片
		cv::Mat img = cv::imread((char*)imgs[i].c_str());

		//由于在实际应用中，我使用opencv进行图像加载，所以这里计时的时候统计了
		//图像数据转换的时间，实际上，统计检测时间即可
		pTimer->Reset();//计时器清零

		//将图像转换成dlib支持的图像数据格式
		dlib::cv_image<dlib::bgr_pixel> dlibIm(img);

		// 使用dlib进行人脸检测
		std::vector<dlib::rectangle> faces = hogFaceDetector(dlibIm);

		double dt = pTimer->GetTime();//统计检测用时

		//输出结果文件，方便日后统计，按照文件名，检测用时，人脸数，各个人脸位置按行输出。
		of << imgs[i] << " " << dt << "s " << faces.size() << " ";
		//显示检测结果
		for (int j = 0; j < faces.size(); j++)
		{
			cv::Rect rt;
			rt.x = faces[j].left();
			rt.y = faces[j].top();
			rt.width = faces[j].width();
			rt.height = faces[j].height();
			cv::rectangle(img, rt, cv::Scalar(0, 0, 255), 2);
			of << rt << " ";
		}
		of << std::endl;

		cv::imshow("Dlib-hog", img);
		int key = cv::waitKey();
		if (key == 's')
			break;
	}
	of.close();
	delete pTimer;
	pTimer = 0;
	return 0;
}
```

## 3 检测效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813095550758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813095601725.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813095609290.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813095617758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)


## 4 分析

- 对小人脸检测效果不佳，因为DLIB自带的检测器是按照最小人脸80*80的大小训练的。
