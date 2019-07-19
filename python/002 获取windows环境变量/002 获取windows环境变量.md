# 【Python】-002 获取windows环境变量

&emsp;&emsp;上次在计算机上安装CTex试了一试，安装倒是顺利完成了，但系统中其他程序有一些运行不成功了，排查之后发现系统中PATH环境变量在安装CTex之后只保留了CTex的环境变量，其他的都没了。因此，考虑在安装之前备份所有的环境变量。

[TOC]

## 1、批处理方式

&emsp;&emsp;使用批处理命令set可以保存所有环境变量。
&emsp;&emsp;例如，我想讲所有的环境变量保存到`d:\env.txt`,只需要执行
```
set>>d:\env.txt
```
&emsp;&emsp;即可。

## 2、Python方式

&emsp;&emsp;python中，os模块的`os.environ`可以返回所有的环境变量值。
&emsp;&emsp;在python中执行如下脚本即可：
```python
#coding:utf-8   #强制使用utf-8编码格式
import os

env_dist = os.environ # environ是在os.py中定义的一个dict environ = {}


# 打印所有环境变量，遍历字典
for key in env_dist:
    print(key + ' : ' + env_dist[key])
    with open(r'd:\env.txt', 'a+') as f:
        f.write(key + ' : ' + env_dist[key] + '\n')
```

&emsp;&emsp;以上两种方法获得的环境变量值都是一致的。
