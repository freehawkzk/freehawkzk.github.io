# 【图像处理】 -040 Dlib 深度人脸检测

[toc]

## 1 简介

&emsp;&emsp;Dlib中实现的深度人脸检测是基于MMOD（Maximum-Margin Object Detector(MMOD))，CNN结构。

## 2 使用dlib实现深度人脸检测

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
#include <dlib/image_processing.h>
#include <dlib/dnn.h>
#include <dlib/data_io.h>

// Network Definition
/////////////////////////////////////////////////////////////////////////////////////////////////////
template <long num_filters, typename SUBNET> using con5d = dlib::con<num_filters, 5, 5, 2, 2, SUBNET>;
template <long num_filters, typename SUBNET> using con5 = dlib::con<num_filters, 5, 5, 1, 1, SUBNET>;

template <typename SUBNET> using downsampler = dlib::relu<dlib::affine<con5d<32, dlib::relu<dlib::affine<con5d<32, dlib::relu<dlib::affine<con5d<16, SUBNET>>>>>>>>>;
template <typename SUBNET> using rcon5 = dlib::relu<dlib::affine<con5<45, SUBNET>>>;

using net_type = dlib::loss_mmod<dlib::con<1, 9, 9, 1, 1, rcon5<rcon5<rcon5<downsampler<dlib::input_rgb_image_pyramid<dlib::pyramid_down<6>>>>>>>>;

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


int main(int argc, char** argv)
{
    if (argc < 3)
    {
        std::cout << "Please use this exe like this:" << std::endl;
        std::cout << "OpenCV_Harrx.exe imglist.txt outputpath" << std::endl;
        system("pause");
    }
    std::string imglistfile(argv[1]);
    std::string outputpath(argv[2]);

    std::string mmodModelPath = "./models/mmod_human_face_detector.dat";
    net_type mmodFaceDetector;
    dlib::deserialize(mmodModelPath) >> mmodFaceDetector;

    std::cout << "load" << std::endl;
    //加载待检测的图片路径列表
    std::vector<std::string> imgs = ReadImgList(imglistfile);

    //创建计时器对象
    char* pTimerName = (char*)"Dlib-MMOD";
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

        int frameHeight = img.rows;
        int frameWidth = img.cols;
        int inHeight = 300;
        int inWidth = 0;
        if (!inWidth)
            inWidth = (int)((frameWidth / (float)frameHeight) * inHeight);

        float scaleHeight = frameHeight / (float)inHeight;
        float scaleWidth = frameWidth / (float)inWidth;

        cv::Mat frameDlibMmodSmall;
        resize(img, frameDlibMmodSmall, cv::Size(inWidth, inHeight));
        // Convert OpenCV image format to Dlib's image format
        dlib::cv_image<dlib::bgr_pixel> dlibIm(frameDlibMmodSmall);
        dlib::matrix<dlib::rgb_pixel> dlibMatrix;
        dlib::assign_image(dlibMatrix, dlibIm);

        // Detect faces in the image
        std::vector<dlib::mmod_rect> faceRects = mmodFaceDetector(dlibMatrix);
        std::vector<cv::Rect> faces;
        for (size_t i = 0; i < faceRects.size(); i++)
        {
            int x1 = (int)(faceRects[i].rect.left() * scaleWidth);
            int y1 = (int)(faceRects[i].rect.top() * scaleHeight);
            int x2 = (int)(faceRects[i].rect.right() * scaleWidth);
            int y2 = (int)(faceRects[i].rect.bottom() * scaleHeight);
            cv::Rect face;
            face.x = x1;
            face.y = y1;
            face.width = x2 - x1;
            face.height = y2 - y1;
            faces.push_back(face);
        }

        double dt = pTimer->GetTime();//统计检测用时

        //输出结果文件，方便日后统计，按照文件名，检测用时，人脸数，各个人脸位置按行输出。
        of << imgs[i] << " " << dt << "s " << faces.size() << " ";
        std::cout << imgs[i] << " " << dt << "s " << faces.size() << " ";
        //显示检测结果
        for (int j = 0; j < faces.size(); j++)
        {
            cv::rectangle(img, faces[j], cv::Scalar(0, 0, 255), 2);
            of << faces[j] << " ";
            std::cout << faces[j] << " ";
        }
        of << std::endl;
        std::cout << std::endl;
        cv::imshow("Dlib-MMOD", img);
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

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813132335937.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813132340628.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813132344558.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=,size_16,color_FFFFFF,t_70)

## 4 分析

- 对小人脸检测效果不佳。
- 在GPU上实现速度极快，约5ms一张图，但CPU上实现检测速度极慢，>1.5s一张。
- 检测精度有待提高，需要重新训练模型。
