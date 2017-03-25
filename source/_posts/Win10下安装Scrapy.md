---
title: Win10下安装Scrapy
date: 2017-01-16 15:45:59
categories: 爬虫
tags:
  - 爬虫
  - Scrapy
---

之前在学校的Win8下面按照官方文档一路顺风顺水, 自己Win10的电脑装起来就磕磕绊绊, Windows系统对开发还是不友好呢.
以及安装的时候尽量全程不要有中文路径, 避免奇怪的bug.

---

## 安装Python

从[python官网](https://www.python.org/downloads/)下载合适的版本安装(我这里下的是python 2.7.13 amd64 for windows).
安装的时候选择自动加入环境变量, 或者装完后手动将下面的路径加到环境变量`Path`中:

`C:\Python27\;C:\Python27\Scripts\;`

安装成功后可以在控制台查看python版本.

`python --version`
<!-- more -->

## 安装pywin32

下载对应版本的[pywin32](https://sourceforge.net/projects/pywin32/files/)并安装, 我下载的是`build 220 amd64`.

安装时直接双击或用管理员身份运行都会得到以下错误信息:

```
close failed in file object destructor:
sys.excepthook is missing
lost sys.stderr
```

正确的做法是用管理员身份打开cmd, 在用命令行启动对应的安装文件.

可以在python环境下`import win32com`, 没有报错则安装成功.

```
python
>>> import win32com
```

## 安装lxml

如果直接用pip安装Scrapy, 会得到以下信息:

`Could not find function xmlCheckVersion in library libxml2. Is libxml2 installed?`

因为pip在安装lxml的时候需要依赖libxml2和libxslt. 这里我直接使用[unofficial Windows binaries](http://www.lfd.uci.edu/~gohlke/pythonlibs/#lxml)下载安装lxml, 这样就无需安装libxml2和libxslt了.

将下载的`xxx.whl`用pip安装:

`pip install xxx.whl`

## 用pip安装Scrapy

`pip install Scrapy`

安装成功后可以查看Scrapy的版本

`scrapy version`

## 其他注意事项

路径最好不要有中文名, 如果出现类似`ascii`字样的报错, 估计就是中文路径在解析时出错了.

不要用virtualenv, 因为pywin32会去识别python的安装目录, 而不会装到virtualenv里面(当然有可能是我不会装...), 到时候高高兴兴以为装好了, 一跑scrapy shell发现提示没有pywin32就傻眼了.

`scrapy shell 'http://quotes.toscrape.com/page/1/'`

---

参考:
[Scrapy Installation Guide](https://doc.scrapy.org/en/latest/intro/install.html)
[python安装lxml，在windows环境下](http://blog.csdn.net/g1apassz/article/details/46574963)
[Vista以上系统安装RobotFramework的注意事项(python安装pywin32也遇到了)](http://blog.csdn.net/frankarmstrong/article/details/8949987)
