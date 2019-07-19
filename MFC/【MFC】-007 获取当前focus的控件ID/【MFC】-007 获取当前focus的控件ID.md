# 【MFC】-007 获取当前focus的控件ID

[TOC]

## 1、由来

&emsp;&emsp;通过tab键切换控件输入焦点时，需要根据不同控件处理不同的按键消息。因此，需要知道当前输入焦点所在的控件的ID。

## 2、实现

&emsp;&emsp;通过`GetFocus`来获得当前输入焦点所在窗口，再获取其控件ID。

```C++
		CWnd* pWnd = GetFocus();// Get current foucs window
		if (pWnd)
		{
			int nID = pWnd->GetDlgCtrlID();// get the window's Ctrl ID.
			if (nID != IDC_BTN_EXIT
				&& nID != IDC_BTN_START
				&& nID != IDC_BTN_STOP
				&& nID != IDC_BTN_RECENTER)
			{// according to the ctrl ID, do something you want
				return TRUE;
			}
		}
```
