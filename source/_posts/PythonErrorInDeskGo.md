---
title: 在腾讯桌面整理上使用Python遇到的坑
comments: true
mathjax: false
date: 2022-04-08 21:30:08
updated: 2022-04-08 21:30:08
tags: ["Python","程序开发"]
---

最近用Python写了个小的GUI程序，用`Pyinstaller`打包之后，准备拖到桌面试用，结果竟然抛了异常，顿时感到很奇怪。因为按道理来说这个异常已经在代码里处理过了，于是开始排查问题。

<!-- more -->

## 背景

这段时间疯狂在网上看剧，下载的剧名总是乱七八糟的，而我使用的播放器又只会将名字大致相同的视频一同加入播放列表。为了能方便自动播放下一集，同时也方便做归档，于是便有了批量将文件改名的需求。

由于系统自带的批量重命名实在不尽如人意，具体表现为文件名后的序号会自带括号，非常有碍观瞻（强迫症了属于是），所以又去网上找了一下现成的，要么界面过于复杂，要么就是缺少所需的功能，于是自己用Python整了一个。

根据之前的经验，如果使用自带的`tkinter`库制作图形界面会相当丑，于是选用了`PySide6`进行GUI的制作。

偷偷打个广告：[批量重命名助手-项目地址-Github](https://github.com/Reilkay/file-rename-gui)

## 遇到的问题

如同前言中所说，在桌面双击打开该程序时，程序抛出了异常，报错信息如下：

```bash
Traceback (most recent call last):
  File "utils\config.py", line 17, in get
  File "toml\decoder.py", line 133, in load
FileNotFoundError: [Errno 2] No such file or directory: './config.toml'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "main.py", line 7, in <module>
  File "window\main_window.py", line 28, in __init__
  File "utils\config.py", line 20, in get
  File "utils\config.py", line 39, in init_config_file
PermissionError: [Errno 13] Permission denied: './config.toml'
```

然而，我是处理过这个异常的（事实上相当于我是在故意利用程序抛出的异常，来检测是否有配置文件存在，然后对配置文件进行一个初始化），该部分代码如下：

```python
def get(self) -> dict:
    try:
    	config = toml.load(self.__path)
    except FileNotFoundError:
    	self.init_config_file()
    	config = toml.load(self.__path)
    return config
```

于是我重新看了下程序抛出的异常，发现了下面这个部分：

```bash
During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "main.py", line 7, in <module>
  File "window\main_window.py", line 28, in __init__
  File "utils\config.py", line 20, in get
  File "utils\config.py", line 39, in init_config_file
PermissionError: [Errno 13] Permission denied: './config.toml'
```

原来是处理异常的过程中又抛出了`PermissionError`的异常，那么为什么明明是在桌面上还会有没有权限的错误呢？

## 问题原因探索

经过长时间的百思不得其解之后，我偶然在文件资源管理器中打开了桌面的路径，并在此处执行了打包生成的`.exe`文件。

震惊！并没有出现任何错误，配置文件也十分正常的生成了。

这时我才终于意识到是我先前为了整理桌面安装的“腾讯桌面整理”导致的问题，随后我又使用该软件的文件映射功能，将D盘上的一个文件夹（D:/test）映射到桌面上进行测试。结果在同样的路径下，用腾讯桌面整理打开该程序则会抛出`PermissionError`的异常，而在文件夹中打开则不会有问题。终于发现了导致这个问题的元凶就是腾讯桌面整理，那么为什么会出现这种问题呢？我又接着写了一个小程序来探究。

 ```python
class MainWindow(QMainWindow, Ui_MainWindow):

    def __init__(self, parent=None):
        super().__init__(parent)
        self.setupUi(self)

        self.setWindowTitle('测试')
        self.argv.setText(sys.argv[0])
        self.cwd.setText(os.getcwd())
        self.abspath.setText(os.path.abspath('.'))
 ```

这里分别使用了三种方式来获取当前程序运行的路径，最终得到的结果如下：

+ 正常在文件夹中开启

  ![image.png](https://s2.loli.net/2022/04/08/ROQcXiu8KTrE1CU.png)

+ 在腾讯桌面整理中打开

  ![image.png](https://s2.loli.net/2022/04/08/sUxNSLHD7EYTZQh.png)

使用`Process Expolorer`查看该测试程序（在腾讯桌面整理中打开）的路径，也得出同样的结果，如下图所示：

![image.png](https://s2.loli.net/2022/04/08/9fA6JSqdTYWPcQz.png)

顺着当前路径，最终是找到了腾讯桌面整理的安装路径下。至此，就可以理解出错的原因，由于程序的当前工作路径在`C:\Program Files (x86)`下，所以并没有新建文件的权限。

## 具体原因分析

这里也想了很久也不明白为什么，腾讯桌面整理的原理到底是什么？在网上查了查后，我大致猜想如下：

腾讯桌面整理劫持了系统自带的桌面（或在上方覆盖了一层？），当我在桌面上双击打开应用是，是由腾讯桌面整理的程序启动了我要打开的程序。这样就导致了程序的当前路径和启动路径不一致（例如：由`SOME PATH/A/a.exe`启动了`SOME PATH/B/b.exe`，那么，`b.exe`的当前路径就为`SOME PATH/A`，但启动路径仍为`SOME PATH/B`）

所以总结来说，在应用编写时，更好的选择是使用程序的启动路径，以避免在相互调用时出现的”灵异现象“。