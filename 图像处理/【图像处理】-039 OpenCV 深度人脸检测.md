# 【图像处理】 -039 OpenCV深度人脸检测

[toc]

## 1 简介

&emsp;&emsp;深度学习是当前的大热门啊，OpenCV在3.3版本之后就有了DNN模块，可以用这个模块来运行训练好的深度学习模型，进行相关网络的使用。
&emsp;&emsp;对于人脸检测，OpenCV的DNN模块，提供一种基于SSD的检测方案，使用ResNet-10作为后端。模型中自带了两种，一种是基于Caffe的float 16模型5.4MB，这一种是基于Tensorflow的int8模型2.7MB。

## 2 使用OpenCV DNN模块人脸检测

```C++
// Dlib_HOG.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//

#include <iostream>
#include <string>
#include <fstream>
#include "opencv2/opencv.hpp"
#include "../OpenCV_Harr/OpenCV_Harr/HighPerformanceTimer.hpp"


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

    // 模型路径
    const std::string caffeConfigFile = "./models/deploy.prototxt";
    const std::string caffeWeightFile = "./models/res10_300x300_ssd_iter_140000_fp16.caffemodel";

    const std::string tensorflowConfigFile = "./models/opencv_face_detector.pbtxt";
    const std::string tensorflowWeightFile = "./models/opencv_face_detector_uint8.pb";

    // SSD框架，输入大小是300*300
    const size_t inWidth = 300;
    const size_t inHeight = 300;
    const double inScaleFactor = 1.0;
    const float confidenceThreshold = 0.7;//置信系数阈值
    const cv::Scalar meanVal(104.0, 177.0, 123.0);

    //创建深度网络，并加载Caffe预训练模型
    cv::dnn::Net net = cv::dnn::readNetFromCaffe(caffeConfigFile, caffeWeightFile);

    //加载待检测的图片路径列表
    std::vector<std::string> imgs = ReadImgList(imglistfile);

    //创建计时器对象
    char* pTimerName = (char*)"OpenCV-DNN";
    CHighPerformanceTimer* pTimer = new CHighPerformanceTimer(pTimerName, 11, true);

    std::ofstream of(outputpath, std::ofstream::out);
    //循环处理所有图片
    for (int i = 0; i < imgs.size(); i++)
    {
        //读取图片
        cv::Mat img = cv::imread((char*)imgs[i].c_str());

        //由于在实际应用中，我使用opencv进行图像加载，所以这里计时的时候统计了
        //图像数据转换的时间，实际上，统计检测时间即可
        pTimer->Reset();//计时器清零


        int frameHeight = img.rows;
        int frameWidth = img.cols;
        // 将输入图像转换成网络输入blob
        cv::Mat inputBlob = cv::dnn::blobFromImage(img, inScaleFactor, cv::Size(inWidth, inHeight), meanVal, false, false);

        // 设置网络输入
        net.setInput(inputBlob, "data");
        cv::Mat detection = net.forward("detection_out");//获取网络输出

        // 输出后处理
        cv::Mat detectionMat(detection.size[2], detection.size[3], CV_32F, detection.ptr<float>());

        std::vector<cv::Rect> faces;
        for (int i = 0; i < detectionMat.rows; i++)
        {
            float confidence = detectionMat.at<float>(i, 2);

            if (confidence > confidenceThreshold)
            {
                int x1 = static_cast<int>(detectionMat.at<float>(i, 3) * frameWidth);
                int y1 = static_cast<int>(detectionMat.at<float>(i, 4) * frameHeight);
                int x2 = static_cast<int>(detectionMat.at<float>(i, 5) * frameWidth);
                int y2 = static_cast<int>(detectionMat.at<float>(i, 6) * frameHeight);
                cv::Rect face;
                face.x = x1;
                face.y = y1;
                face.width = x2 - x1;
                face.height = y2 - y1;
                faces.push_back(face);//获取检测到的人脸框
                std::cout << face << std::endl;
            }
        }

        double dt = pTimer->GetTime();//统计检测用时

        //输出结果文件，方便日后统计，按照文件名，检测用时，人脸数，各个人脸位置按行输出。
        of << imgs[i] << " " << dt << "s " << faces.size() << " ";
        std::cout << imgs[i] << " " << dt << "s " << faces.size() << " ";
        //显示检测结果
        for (int j = 0; j < faces.size(); j++)
        {
            cv::rectangle(img, faces[j], cv::Scalar(0, 255, 0), 2, 4);
            of << faces[j] << " ";
            std::cout << faces[j] << " ";
        }
        of << std::endl;
        std::cout << std::endl;
        cv::imshow("OpenCV-DNN", img);
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

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813105921372.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813105926392.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813105932962.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813105937754.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813105945495.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813105949986.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813105955387.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)

## 4 分析

- 在CPU上可以达到实时。
- 对人脸尺寸没有太多要求。
- 检测准确率高。
- 可以检测正脸、测量、旋转等脸部。
- 对脸部遮挡之后的检测结果也挺高。
