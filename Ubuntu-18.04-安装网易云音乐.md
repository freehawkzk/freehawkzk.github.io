# Ubuntu 18.04 安装网易云音乐

[TOC]

## 1 安装

&emsp;&emsp;最近将日常使用系统从windows切换到Ubuntu18.04 LTS，在Windows下使用网易云音乐听歌挺习惯的，因此希望在Ubuntu下也能使用。网上看到网易提供了linux下的网易云音乐客户端，大为欣慰。

&emsp;&emsp;在官网下了网易云音乐安装包之后，通过`sudo dpkg -i `安装了之后，发现在软件中能看到网易云音乐的图标，可是点击运行之后却没有任何窗口出现。

## 2 启动

&emsp;&emsp;网上放狗搜了之后，很多人都有这个问题。偶尔试了使用sudo运行网易云音乐，发现窗口奇迹般的出现了。于是开始每次在终端中输入`sudo netease_cloud_music`来运行的日子。

&emsp;&emsp;这种方法虽然能够使网易云音乐运行，但每次都需要打开一个终端不能关闭，让有轻微强迫症的我看了觉得很不舒服。

&emsp;&emsp;既然在终端中输入命令能够运行，那肯定可以通过脚本的方式运行。

&emsp;&emsp;于是我在`~`目录下新建了一个脚本`.netease.sh`,该脚本的内容如下：
```
#!/bin/bash
echo "mypassword" | sudo -S netease-cloud-music &
sleep 0.1
exit
```
&emsp;&emsp;然后使用`chmod`命令，给脚本增加可执行权限。

&emsp;&emsp;接下来，我编辑`~/.bashrc`，在末尾增加了一行：
```
alisa netease=". .netease.sh"
```
&emsp;&emsp;注意，上一行的两个点之间的空格。

&emsp;&emsp;之后，重启计算机，即可在终端中输入`netease`命令来运行网易云音乐了。
