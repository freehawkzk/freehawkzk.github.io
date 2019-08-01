# 【Python】-007 删除VS编译后生成的临时文件

&emsp;&emsp;作为一个曾经主攻word的伪C++工程师，每次使用VS编译C++工程之后发现总会生成一堆各种各样的临时文件，虽然眼不见为净，但每次收到别人给我发的带有一堆临时文件的项目源代码的时候，内心总会有一万只神兽呼啸而过。给代码之前就不会清理一下么。。。

&emsp;&emsp;虽然windows的批处理也能很好的按照特定的后缀名删除文件，但批处理我实在是不熟悉，所以只能用python来干这活了。

## 1 编写Python脚本递归的删除目录下的特定类型文件

```python
'''
递归的删除目录下特定后缀名的文件，如果删除文件后文件夹为空，删除该文件夹
'''

import os
import sys

dir=[]
if(len(sys.argv) == 1):
    print("You dont input the folder path that you want to list file name, dir will be default to ./!\n")
    dir="./"
else:
    dir=sys.argv[1]

# 指定VS临时文件
tobecleanedfileexts=[".idb",".db",".ipdb",".pdb",".iobj",".ilk",".exp",".obj",".log",".tlog",".lastbuildstate",".opendb",".ipch",".pch","unsuccessfulbuild"]

dir = dir.replace("\\\\",'/')
dir = dir.replace("\\",'/')
path = dir #文件夹目录
datas = []

def CleanPath(filepath):
    fileNames = os.listdir(filepath)  # 获取当前路径下的文件名，返回List
    for file in fileNames:
        newDir = filepath + '/' + file # 将文件命加入到当前文件路径后面
        if os.path.isfile(newDir):  # 如果是文件
            fileext = os.path.splitext(file)[-1]
            if fileext in tobecleanedfileexts :
                newDir = newDir.replace("\\\\", '/')
                newDir = newDir.replace("\\", '/')
                try:
                    os.remove(newDir)
                    print("remove "+newDir)
                except PermissionError as identifier:
                    pass

        else:
            CleanPath(newDir)                #如果不是文件，递归这个文件夹的路径
            try:
                if not os.listdir(newDir):
                    os.removedirs(newDir)
                    print("removedir "+newDir)
            except FileNotFoundError as identifier:
                pass

if __name__ == "__main__":
    CleanPath(path)
```

## 2 让VS编译之后自动执行脚本

&emsp;&emsp;脚本写完了，在Python里试了一下，确实能够删除特定类型的文件了。但每次编译完之后都需要手动去命令行里执行一下，有点费事。作为一个“懒惰促进社会进步”观念的信徒，这活必须让VS帮我干了。

&emsp;&emsp;所以，打开VS，项目属性->生成事件->后期生成事件,在命令行里输入“python CleanVSTempFiles.py脚本路径 需要清理的路径”，这样每次生成完成之后，VS会自动执行这个脚本去清理特定的路径了。

&emsp;&emsp;清理工作是做完了，但在debug模式下需要调试的时候发现刚才清理的时候把所有调试的临时文件也删除了。于是将刚才的命令行只设置到release模式下。恩，挺好了。每次release模式编译完之后进行清理。
