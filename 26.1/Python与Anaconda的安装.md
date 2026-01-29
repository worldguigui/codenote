# Python与Anaconda的安装

## 一.PyCharm的安装
PyCharm 是一个强大的 Python 集成开发工具，搜索 PyCharm 官网，直接下载安装即可。<br/>

## 二.Anaconda的安装
Anaconda 是一个强大的 Python 包管理工具，The Most Trusted Distribution for Data Science。可以帮助我们很方便地安装、删除 Python 包；配置不同的虚拟环境。<br/>

搜索 Anaconda 官网，直接下载安装即可。<br/>

安装时注意不要安装在 C 盘。<br/>

### 1.配置环境变量
在系统环境变量的 Path 目录下，新增几个路径。<br/>

```
E:\python\anaconda
E:\python\anaconda\Scripts
E:\python\anaconda\Library\bin
E:\python\anaconda\Library\mingw-w64\bin
```

### 2.修改依赖安装路径
在 C:\Users\用户名\.condarc 中修改。<br/>

删掉所有原有内容，修改为：（注意调整目录路径和缩进为两个空格）<br/>
```
envs_dirs:
  - E:\python\anaconda_envs\envs
pkgs_dirs:
  - E:\python\anaconda_envs\pkgs
```

### 3.修改镜像源
在 C:\Users\用户名\.condarc 中修改。<br/>

具体的镜像源可以直接搜索，复制粘贴即可。<br/>

由于我下载的并不慢，所以在用的时候这些配置都删掉了，有没有用就不好说了（反正人家网页就这么给的）。<br/>
```
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```
