# 【图像处理】 -037 OpenCV人脸检测

&emsp;&emsp;最近，工作中需要对输入图像进行人脸检测，因此，花了点时间来对目前市面上的人脸检测技术进行了一次初步测试。这里进行简单记录。

[toc]

## 1 介绍

&emsp;&emsp;在2001年之前，人脸检测还主要是基于人脸特征的，因此，OpenCV中自带的基于haar 级联分类器的人脸检测器常年处于当时的state-of-art的水平。OpenCV自带的经过训练的模型主要要正脸、侧脸、人眼、上半身等分类器。这里，我主要尝试实现使用OpenCV的正脸检测器。

## 2 实现

```C++
// OpenCV_Harr.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//

#include <iostream>
#include <string>
#include <fstream>
#include "opencv2/opencv.hpp"
#include "HighPerformanceTimer.hpp"//windows平台高性能计时器

//读取待检测文件列表
std::vector<std::string> ReadImgList(std::string& imglistfilename)
{
	std::vector<std::string> imgs;
	std::ifstream imglistfile(imglistfilename, std::ifstream::in);
	std::string line;
	while (getline(imglistfile,line))//按行读取
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


	//检测器对象
	cv::CascadeClassifier frontface;
	//加载模型文件
	bool hr = frontface.load("./models/haarcascade_frontalface_alt.xml");
	if (!hr)
	{	
		//加载失败
		std::cout << "Load ./models/haarcascade_frontalface_alt.xml failed!" << std::endl;
		system("pause");
	}

	//加载待检测的图片路径列表
	std::vector<std::string> imgs = ReadImgList(imglistfile);

	//创建计时器对象
	char* pTimerName = (char*)"OpenCV-Harr";
	CHighPerformanceTimer* pTimer = new CHighPerformanceTimer(pTimerName, 12, true);
	
	std::ofstream of(outputpath, std::ofstream::out);
	//循环处理所有图片
	for (int i = 0;i< imgs.size();i++)
	{
		//读取图片
		//OpenCV正脸检测处理的是灰度图，读取之后转换成灰度图
		cv::Mat img = cv::imread((char*)imgs[i].c_str());
		cv::Mat gray;
		cv::cvtColor(img, gray, cv::COLOR_BGR2GRAY);

		std::vector<cv::Rect> faces;
		
		pTimer->Reset();//计时器清零

		frontface.detectMultiScale(img, faces);//进行人脸检测
		
		double dt = pTimer->GetTime();//统计检测用时

		//输出结果文件，方便日后统计，按照文件名，检测用时，人脸数，各个人脸位置按行输出。
		of << imgs[i] << " " << dt << "s " << faces.size() << " ";
		//显示检测结果
		for (int j = 0;j<faces.size();j++)
		{
			cv::rectangle(img, faces[j], cv::Scalar(0, 0, 255), 2);
			of << faces[j] << " ";
		}
		of << std::endl;

		cv::imshow("OpenCV-harr", img);

		int key = cv::waitKey();
		if(key == 's')
			break;
	}
	of.close();
	delete pTimer;
	pTimer = 0;
	return 0;
}
```

## 3 检测效果
### 3.1 检测成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190812172331647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190812172439608.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190812172604522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
### 3.2 检测失败
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190812172418649.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190812172513689.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190812172533716.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)

## 4 分析

- 在CPU上基本可以做到实时；
- 对人脸存在一定角度倾斜的时候，检测准确度会降低；
- 存在误检的情况。
