# 【MFC】-005 根据进程名获取进程ID

[TOC]

## 1、由来

&emsp;&emsp;在使用TrackIR5进行位姿跟踪时，使用网上的方法进行姿态数据获取时，需要确保TrackIR5本身的软件一直处于运行状态。否则会获取不到数据。因此，在程序运行前需要判断TrackIR5软件是否已经处于运行状态。

## 2、实现

&emsp;&emsp;每一个进程在运行过程中，在系统中都会有一个独一无二的进程ID，该ID在各次运行时并不相同，除非经过特殊处理，进程名称每次都是一样的。因此，可以考虑从进程名称去获取进程的ID。进而进行后续处理。

```C++
DWORD GetProcessIDByName(const wchar_t* pName)
{
	HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
	if (INVALID_HANDLE_VALUE == hSnapshot) {
		return NULL;
	}
	PROCESSENTRY32 pe = { sizeof(pe) };
	for (BOOL ret = Process32First(hSnapshot, &pe); ret; ret = Process32Next(hSnapshot, &pe)) 
	{
		if (wcscmp(pe.szExeFile, pName) == 0)
		{
			CloseHandle(hSnapshot);
			return pe.th32ProcessID;
		}
	}
	CloseHandle(hSnapshot);
	return 0;
}
```

## 3、注意

&emsp;&emsp;该方法是根据进程的exe文件名进行查找的，而进程文件名与其在windows任务管理器中显示的名称可能不一致！