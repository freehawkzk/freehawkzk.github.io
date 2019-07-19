# 001- VS2015 error LNK2001 无法解析的外部符号 _WinMain@16

[TOC]

## 1、现象


&emsp;&emsp;使用VS2015新建控制台空项目，之后往项目中手动添加cpp文件，编译时出现`error LNK2001 无法解析的外部符号 _WinMain@16`错误，链接错误。

## 2、解决方法

&emsp;&emsp; (1) 项目属性->C/C++->预处理器，预处理器定义中删除`_WINDOWS`,添加`_CONSOLE`;
&emsp;&emsp; (2) 项目属性->链接器->系统，子系统选项选择为`控制台 (/SUBSYSTEM:CONSOLE)`;