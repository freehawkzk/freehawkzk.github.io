# 【GDI+】2048游戏
&emsp;&emsp;2048游戏在今年年初的时候风靡天下，作为一个程序员，在玩别人写的游戏的同时也想着用自己的方式来实现一下。由于只是做一个简单Demo,所以选择了MFC配合GDI+的方式来实现。
[TOC]
## 1、游戏规则
&emsp;&emsp;2048游戏是前段时间很火的一款小游戏，席卷P端，安卓和IOS。实际上这款游戏的规则比较简单。主要规则如下:
- 1 游戏区域内随机出现分值为2或4的方块
- 2 可以向上下左右四个方向进行操作
- 3 操作时，所有方块会向对应方向移动，如果遇到与自己分数相同的块，两者会合并成1个块
- 4 当游戏区域被填满时游戏结束
- 5 游戏开始后间隔固定时间出现新块

## 2、GDI+基础
&emsp;&emsp;本文中使用VS2015 MFC + GDI+ 进行软件界面的开发。(丑点，但忍了吧)。新建基于对话框的MFC程序，使用Unicode编码x86编译。接下来介绍实验中使用到的GDI+的一些使用方式。
### 2.1 GDI+初始化
&emsp;&emsp;GDI+在使用之前，需要进行初始化。在MFC中使用时，根据网上普遍流传的使用方式来说，需要在CxxApp的头文件中增加成员变量：
```C++
Gdiplus::GdiplusStartupInput gdiplusStartupInput;
ULONG_PTR gdiplusToken;
```
&emsp;&emsp;在CxxApp的`InitInstance`成员函数中，添加`Gdiplus::GdiplusStartup(&gdiplusToken, &gdiplusStartupInput, NULL);`函数调用，注意，该调用一定要在窗口显示之前。
### 2.2 GDI+画矩形方框
&emsp;&emsp;GDI+绘制矩形方框，主要通过`Gdiplus::Graphics`的成员函数`DrawRectangle`函数进行。其中`Gdiplus::Graphics`可以通过传入需要绘制的DC的指针进行新建。**注意**，使用`GetDC()`函数获取对应的DC使用完毕之后，需要使用`ReleaseDC()`函数释放掉。
```C++
CDC* pDC = GetDC();
Gdiplus::Graphics g(pDC->m_hDC);
Gdiplus::Pen pen(g_colorBordLine,g_fBordLineWidth);
RECT rt;
GetClientRect(&rt);
float fGameAreaWidth = (rt.right - rt.left)*0.8f;
float fGameAreaHeight = rt.bottom - rt.top;
Gdiplus::Rect gameRect(g_fBordLineWidth / 2.0, g_fBordLineWidth / 2.0, fGameAreaWidth - g_fBordLineWidth, fGameAreaHeight - g_fBordLineWidth);
g.DrawRectangle(&pen, gameRect);
ReleaseDC(pDC);
```
&emsp;&emsp;调用`DrawRectangle`可以通过指定绘制的画笔和需要绘制的区域来在指定区域绘制矩形方框。其中画笔可以有颜色参数和线宽参数。
### 2.3 GDI+填充矩形
&emsp;&emsp;GDI+中填充矩形主要用于绘制矩形方块。在2048游戏中，大部分时间都是在方块的填充中完成的。通过调用`FillRectangle`函数来完成指定区域的填充。`g.FillRectangle(&brush, gameRect);`,参数包括一个画刷和一个矩形区域。GDI+使用代码指定的画刷填充第二个参数指定的矩形区域。
### 2.4 GDI+显示文字
&emsp;&emsp;文字的显示在GDI+中通过`DrawString`函数完成，该函数调用时需要指定输出字符串的内容、长度、字体、开始位置、使用的画刷。
### 2.5 GDI+反初始化
&emsp;&emsp;GDI+使用完成后，在退出之前需要进行反初始化。重载CxxApp的`ExitInstance`函数，在其中调用`Gdiplus::GdiplusShutdown(gdiplusToken);`即可完成反初始化。
## 3、实现
### 3.1 游戏区域的表示
&emsp;&emsp;游戏区域可以通过指定游戏区的行数、列数来指定游戏区中可以容纳的方块的行数和列数，从而指定最大方块数目。游戏区通过一个`std::vector<std::vector<int> >`容器保存当前游戏区中各个方块的状态。vv[i][j]表示第i行第j列的方块，vv[i][j]=0表示该方块为空，没有方块。vv[i][j]=2/4/8/16/32/64/128/256/512/1024/2048表示该方块的当前值。游戏开始时，将vv设置为已经创建好(已加入所有元素，确保vv[i][j]访问的有效性)。
```C++
typedef std::vector<std::vector<int> > vv;
extern vv g_vvData;//保存游戏区域中各块的当前值
```
```C++
void CGame2048Dlg::ResetGameMap()
{
	m_lock.Lock();
	if (g_vvData.size() > 0)
	{
		for each ( auto  v in g_vvData)
		{
			v.clear();
		}
		g_vvData.clear();
	}

	for (int i = 0; i < g_nBlockNumY; i++)
	{
		g_vvData.push_back(std::vector<int>(g_nBlockNumX, 0));
	}
	m_lock.Unlock();
}
```
### 3.2 增加新块
&emsp;&emsp;增加新块主要进行vv中数据的修改，修改之后触发界面更新即可。
```C++
void CGame2048Dlg::AddABlock()//向棋盘中添加一个方块
{
	int nX = 0; 
	int nY = 0;
	do 
	{
		nX = rand() % g_nBlockNumY;
		nY = rand() % g_nBlockNumX;
	} while (g_vvData[nX][nY] != 0);//循环随机获取一个空的块
	
	m_lock.Lock();
	g_vvData[nX][nY] = 2 * (rand() % 2) + 2;//为该空块赋一个随机值，2或4
	m_lock.Unlock();
}
```
&emsp;&emsp;增加新块还需要注意一个逻辑，当前空余的方块数大于等于2时，一次增加两个方块，当只空余一个方块时，增加一个方块。当前没有空余方块时，触发游戏结束。
&emsp;&emsp;目前，设计的是按照固定时间间隔自动添加新块。未来，可以依据分数自动划分难度等级或者依据选择的模式不同，自动调节时间间隔。
### 3.3 向左移动
&emsp;&emsp;这里只介绍向左一个方向移动的移动逻辑。其余方向的移动类似处理。向左移动主要包括两个步骤，(1)从左到右，将行的空格全部排列到行尾；(2)从右到左进行合并，将分值相同的两个块合并。
#### 3.3.1 清理空块
```C++
for (int i = 0; i < g_vvData.size(); i++)
{
	for (int j = 0; j < g_vvData[i].size(); j++)//从左向右搜索
	{
		if (g_vvData[i][j] == 0)//找到第一个等于0的值
		{
			for (int k = j+1; k < g_vvData[i].size();k++)//从这个位置开始向右搜索
			{
				if (g_vvData[i][k] != 0)//找到0右边的第一个不等于0的值
				{
					g_vvData[i][j] = g_vvData[i][k];//将该值与0交换
					g_vvData[i][k] = 0;
					break;//开始下一轮搜索
				}
			}
		}
	}
}
```
#### 3.3.2 合并块
```C++
for (int i = 0; i < g_vvData.size(); i++)
{
	for (int j = g_vvData[i].size()-1; j > 0; j--)//从右向左搜索
	{
		if (g_vvData[i][j] != 0)//找到第一个不等于0的值
		{
			if (g_vvData[i][j-1] == g_vvData[i][j])//如果该值左边一个的值等于该值
			{
				g_vvData[i][j - 1] *= 2;//左边元素值乘以2
				g_vvData[i][j] = 0;//将本值赋为0，
				for (int m = j; m < g_vvData[i].size()-1; m++)//将本值右边的所有值向左移动一位
				{
					g_vvData[i][m] = g_vvData[i][m + 1];
				}
				g_vvData[i][g_vvData[i].size() - 1] = 0;//最后一个赋值为0
			}
		}
	}
}
```
### 3.4 游戏区域的绘制
&emsp;&emsp;游戏区域的绘制主要是根据游戏数据绘制响应的颜色块。
```C++
void CGame2048Dlg::DrawBlocks()
{
	CDC* pDC = GetDC();
	Gdiplus::Graphics g(pDC->m_hDC);


	RECT rt;
	GetClientRect(&rt);
	float fWidth = (rt.right - rt.left) * 0.8f;
	float fHeight = rt.bottom - rt.top;

	float fBlockWidth = (fWidth - 2 * g_fBordLineWidth - (g_nBlockNumX + 1)*g_fGapWidth) / g_nBlockNumX;
	float fBlockHeight = (fHeight - 2 * g_fBordLineWidth - (g_nBlockNumY + 1)*g_fGapWidth) / g_nBlockNumY;


	Gdiplus::FontFamily fontFamily(L"楷体");
	Gdiplus::Font myFont(&fontFamily, 40, Gdiplus::FontStyle::FontStyleRegular, Gdiplus::Unit::UnitPoint); //第二个是字体大小
	Gdiplus::StringFormat format;
	format.SetAlignment(Gdiplus::StringAlignment::StringAlignmentCenter);
	for (int i = 0; i < g_nBlockNumY; i++)
	{
		for ( int j = 0; j < g_nBlockNumX; j++)
		{

			if (g_vvData[i][j] != 0)
			{
				Gdiplus::SolidBrush b(g_colorMap[g_vvData[i][j]]);
				Gdiplus::Rect blockRt(g_fBordLineWidth + g_fGapWidth*(j + 1) + j*fBlockWidth, g_fBordLineWidth + g_fGapWidth*(i + 1) + i*fBlockHeight, fBlockWidth, fBlockHeight);
				g.FillRectangle(&b, blockRt);
				Gdiplus::SolidBrush textBrush(g_colorText);
				Gdiplus::PointF pt;
				pt.X = blockRt.GetLeft();
				pt.Y = blockRt.GetTop();
				g.DrawString(g_indexStringMap[g_vvData[i][j]], wcslen(g_indexStringMap[g_vvData[i][j]]), &myFont, pt, &textBrush);
			}
			
		}
	}

	ReleaseDC(pDC);
}
```
### 3.5 游戏分数计算
&emsp;&emsp;得分计算方式为当前合并之后的块分值的2倍，比如，某一次操作将4和4合并，合并后的块为8，那么本次操作的得分为8*2= 16分。
### 3.6 游戏结束的判断
&emsp;&emsp;游戏结束主要由两种情况：(1) 某一个块的得分到达2048；(2) 游戏区全部被填满，无法再添加新块。
```C++
m_nFreePos = 0;
	for each (auto var in g_vvData)
	{
		for each (auto x in var)
		{
			if (x == 0)
			{
				m_nFreePos++;
			}
			if (x == 2048)
			{
				SendMessage(WM_GAME_OVER, 1, 0);
				return true;
			}
		}

	}
	if (m_nFreePos == 0)
	{
		SendMessage(WM_GAME_OVER, 0, 0);
		return true;
	}


	return false;
```
## 4、总结
&emsp;&emsp;只要理清楚游戏规则，这个游戏是比较容易实现的，整个编码过程大概耗时1小时。[代码](https://github.com/freehawkzk/Game2048.git)
&emsp;&emsp;在开发完试完的时候，发现鼠标果然没有键盘来得快啊，使用键盘操作轻松就完成了2048分结束游戏的条件，使用鼠标真是没这么方便。
## 5、效果图
![image](https://img-blog.csdn.net/20180627223550118?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZWVoYXdrems=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
