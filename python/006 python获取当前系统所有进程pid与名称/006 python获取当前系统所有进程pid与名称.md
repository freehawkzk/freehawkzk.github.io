# 【Python】-006 python获取当前系统所有进程pid与名称

&emsp;&emsp;

[TOC]

## 1、实现

&emsp;&emsp;

```Python
import psutil
pids = psutil.pids()
for pid in pids:
    p = psutil.Process(pid)
    print("pid-%d,pname-%s" %(pid,p.name()))
```

## 2、注意

&emsp;&emsp;psutil包需要安装。
```
pip install psutil
```