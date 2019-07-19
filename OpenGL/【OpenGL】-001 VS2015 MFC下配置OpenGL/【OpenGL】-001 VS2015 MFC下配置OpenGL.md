# 【OpenGL】-001 VS2015 MFC下配置OpenGL
&emsp;&emsp;最近在看《OpenGL SuperBiber》，该书示例代码是GLFW+OpenGL实现的，窗口系统采用了GLFW。虽然GLFW是一个优秀的窗口管理系统，但由于我更熟悉MFC，所以希望将该书的代码移植到VS2015 MFC下。
[TOC]

##1、安装GLEW
### 1.1 下载GLEW
&emsp;&emsp;**GLEW**(OpenGL Extension Wrangler Library)是一个用C/C++实现的可移植的OpenGL库。由于微软官方支持的OpenGL版本只持续到1.1，而目前OpenGL已经发展到4.5到4.6版本了。对于1.1之后OpenGL函数必须使用扩展的形式进行支持。
&emsp;&emsp;GLEW下载网址：(https://sourceforge.net/projects/glew/files/glew/2.1.0/)。目前最新版本是2.1.0，可以根据时间从(http://glew.sourceforge.net/) 查询最新版本并下载。
&emsp;&emsp;此处有一小坑，Github上有glew的库，但选择master下载下来之后，在src目录下没有需要的glew.c，感觉有点坑爹。
### 1.2 编译
&emsp;&emsp;GLEW中包含windows下的生成工程文件，在build目录下。使用VS2015打开VC12下的glew.sln，升级编译工具即可顺利编译。
### 1.3 部署
&emsp;&emsp;GLEW编译完之后，根据各人爱好放置需要的.h,.lib..dll即可。网上很多人建议将相关文件丢到system32等系统目录下，而我喜欢将这些放置到当前工程能够通过相对目录访问的目录下，通常我放在解决方案的lib/include目录，将dll放到解决方案的输出目录下，这样，不至于因为单个解决方案污染整个操作系统的相关文件。
##2、MFC下使用OpenGL
&emsp;&emsp;这里，我选择在MFC的对话框程序中使用OpenGL。
###2.1 新建对话框程序
&emsp;&emsp;按照传统套路，一路默认建立MFC对话框程序即可。
###2.2 引入OpenGL头文件和库文件
&emsp;&emsp;在stdafx.h中加入以下代码：
```C++
#include "GL/glew.h"
#ifdef _DEBUG
#pragma comment(lib,"./lib/Debug/Win32/glew32d.lib")
#else
#pragma comment(lib,"./lib/Release/Win32/glew32.lib")
#endif

#include <gl/GL.h>
#include <gl/GLU.h>
#include <gl/glaux.h>

#pragma comment(lib,"OpenGL32.lib")
#pragma comment(lib,"GLu32.lib")
#pragma comment(lib,"GLaux.lib")
```
&emsp;&emsp;注意，**在包含glew.h之前不能包含GL.h**。
###2.3 加入必须的变量
&emsp;&emsp;在对话框中添加成员变量：
```C++
	HGLRC m_hRC;    //RC 绘图上下文
	CDC* m_pDC;        //DC 设备上下文
```
其中m_hRC是OpenGL渲染使用的绘图上下文，m_pDC是渲染窗口的设备上下文。
###2.4 初始化OpenGL
&emsp;&emsp;使用OpenGL进行渲染之前，需要对OpenGL进行初始化。
```C++
BOOL COpenGL3DDlg::InitializeOpenGL()
{
	//客户区获得DC
	m_pDC = new CClientDC(this);
	//Failure to Get DC
	if (m_pDC == NULL)
	{
		MessageBox(L"Error Obtaining DC");
		return FALSE;
	}
	//为DC建立像素格式
	if (!SetupPixelFormat())
	{
		return FALSE;
	}
	//创建 RC
	m_hRC = ::wglCreateContext(m_pDC->GetSafeHdc());
	//Failure to Create Rendering Context
	if (m_hRC == 0)
	{
		MessageBox(L"Error Creating RC");
		return FALSE;
	}
	//设定OpenGL当前线程的渲染环境。
	//以后这个线程所有的OpenGL调用都是在这个hdc标识的设备上绘制。
	if (::wglMakeCurrent(m_pDC->GetSafeHdc(), m_hRC) == FALSE)
	{
		MessageBox(L"Error making RC Current");
		return FALSE;
	}

	GLenum err = glewInit();
	if (GLEW_OK != err)
	{
		/* Problem: glewInit failed, something is seriously wrong. */
		fprintf(stderr, "Error: %s\n", glewGetErrorString(err));
	}

	//背景颜色
	::glClearColor(0.0f, 0.0f, 0.0f, 0.0f);
	//深度缓存 1最大，让任何都能显示出来
	::glClearDepth(1.0f);
	//如果通过比较后深度值发生变化了，会进行更新深度缓冲区的操作
	::glEnable(GL_DEPTH_TEST);
	
```
注意，使用GLEW库的函数之前需要对GLEW进行初始化，调用`glewInit`函数并检查返回值，确定glew库初始化状态。
```C++
BOOL  COpenGL3DDlg::SetupPixelFormat()
{
	static PIXELFORMATDESCRIPTOR pfd =
	{
		sizeof(PIXELFORMATDESCRIPTOR),  // size of this pfd
		1,                              // version number
		PFD_DRAW_TO_WINDOW |            // support window
		PFD_SUPPORT_OPENGL |            // support OpenGL
		PFD_DOUBLEBUFFER,                // double buffered
		PFD_TYPE_RGBA,                  // RGBA type
		24,                             // 24-bit color depth
		0, 0, 0, 0, 0, 0,               // color bits ignored
		0,                              // no alpha buffer
		0,                              // shift bit ignored
		0,                              // no accumulation buffer
		0, 0, 0, 0,                     // accum bits ignored
		16,                             // 16-bit z-buffer
		0,                              // no stencil buffer
		0,                              // no auxiliary buffer
		PFD_MAIN_PLANE,                 // main layer
		0,                              // reserved
		0, 0, 0                         // layer masks ignored
	};
	int m_nPixelFormat = ::ChoosePixelFormat(m_pDC->GetSafeHdc(), &pfd);
	if (m_nPixelFormat == 0)
	{
		return FALSE;
	}
	if (::SetPixelFormat(m_pDC->GetSafeHdc(), m_nPixelFormat, &pfd) == FALSE)
	{
		return FALSE;
	}
	return TRUE;
}
```
###2.5 渲染与界面更新
&emsp;&emsp;在MFC对话框程序中，绘制操作一般都放置于OnPaint函数中，
```C++
void COpenGL3DDlg::OnPaint()
{
	if (IsIconic())
	{
		CPaintDC dc(this); // 用于绘制的设备上下文

		SendMessage(WM_ICONERASEBKGND, reinterpret_cast<WPARAM>(dc.GetSafeHdc()), 0);

		// 使图标在工作区矩形中居中
		int cxIcon = GetSystemMetrics(SM_CXICON);
		int cyIcon = GetSystemMetrics(SM_CYICON);
		CRect rect;
		GetClientRect(&rect);
		int x = (rect.Width() - cxIcon + 1) / 2;
		int y = (rect.Height() - cyIcon + 1) / 2;

		// 绘制图标
		dc.DrawIcon(x, y, m_hIcon);
	}
	else
	{
		//CDialogEx::OnPaint();


		// 清除颜色、深度缓存
		::glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

		//可以添加渲染函数
		g_oglobj.Render();
		// Flush掉渲染流水线
		::glFinish();
		// 交换前后缓存区
		::SwapBuffers(m_pDC->GetSafeHdc());
	}
}
```
在这里调用OpenGL的渲染相关操作。先在BACK缓冲区绘制再交换buffer到前台。
###2.6 背景擦除
&emsp;&emsp;由于窗口的背景由OpenGL进行擦除，因此可以将MFC的GDI擦除背景操作屏蔽，使其直接返回TRUE即可。
```C++
BOOL COpenGL3DDlg::OnEraseBkgnd(CDC* pDC)
{
	// TODO: 在此添加消息处理程序代码和/或调用默认值
	return TRUE;
//	return CDialogEx::OnEraseBkgnd(pDC);
}
```
##3、示例工程下载
下载地址：(https://github.com/freehawkzk/OpenGL3D)