---
title: Windows Terminal美化
comments: true
mathjax: false
date: 2021-03-19 16:34:00
updated: 2021-03-19 16:34:00
tags: ['Windows Terminal','美化']
---



由于去年使用过一段时间的ArchLinux，让我对Linux系统的自由度有了很深的认识，从桌面到系统的各项配置，都可以按照自己的喜好进行配置，像是桌面的各种工具，图标排列，动画，开机界面等等。这次因为一些原因需要在Windows上完成一些工作，恰好又重装了系统，于是就捣鼓了一下Win下的命令行，让它也美观起来~

<!-- more -->

## 预览

![myWinTerminal](https://www.picbed.cn/images/2021/03/23/preview.webp) 

## 安装Windows Terminal

在官方 [README](https://github.com/microsoft/terminal) 中，安装Windows Terminal有如下方法：

+ Microsoft Store（推荐）
+ Github
+ winget
+ Chocolatey
+ Scoop

最简单的方法就是直接打开Store，下载并安装Windows Terminal了。

## Windows Terminal美化

### 主题美化

打开Windows Terminal，在最上方一列点击向下的箭头，点击设置。

> 这里推荐安装vscode用于配置文件的编辑，当然使用自己习惯的编辑器也可。

将主题配置置于”schemes“中：

```json
// Add custom color schemes to this array.
    // To learn more about color schemes, visit https://aka.ms/terminal-color-schemes
	"schemes": []
```

这里分享几个主题网站：

+ [Windows Terminal Themes](https://windowsterminalthemes.dev/) 
+ [TerminalSplash](https://terminalsplash.com/) 

再分享一下自己用的主题：

```json
{
    "name": "Night Owlish Light",
    "background": "#FFFFFF",
    "black": "#011627",
    "blue": "#4876D6",
    "brightBlack": "#7A8181",
    "brightBlue": "#5CA7E4",
    "brightCyan": "#00C990",
    "brightGreen": "#49D0C5",
    "brightPurple": "#697098",
    "brightRed": "#F76E6E",
    "brightWhite": "#989FB1",
    "brightYellow": "#ca9e00",
    "cyan": "#08916A",
    "foreground": "#403F53",
    "green": "#2AA298",
    "purple": "#403F53",
    "red": "#D3423E",
    "white": "#7A8181",
    "yellow": "#9c6c03"
}
```

添加完主题后，在”profiles“项下，将`"colorScheme": "Night Owlish Light"` 置于”defaults“中（应用于全局）

```json
"defaults": {
            // Put settings here that you want to apply to all profiles.
            "colorScheme": "Night Owlish Light",
        },
```

或置于”list“下包含”Windows PowerShell“的块中（应用于PowerShell）即可。

```json
"list": [
            {
                // Make changes here to the powershell.exe profile.
                "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
                "name": "Windows PowerShell",
                "commandline": "powershell.exe",
                "hidden": false,
				"colorScheme": "Night Owlish Light",
            },
```

### 其他设置

> 注意：如果设置透明不生效，可能需要到 Windows 设置->个性化->颜色 中打开 `透明效果` 开关。

同时还可以在这两处配置其他设置，如：

+ 字体(下文配置oh-my-posh需要powerline字体)：`"fontFace": "MesloLGMDZ Nerd Font Mono"`
+ 亚克力效果(半透明)：`"useAcrylic": true`
+ 透明度：`"acrylicOpacity": 0.6`
+ 光标样式：`"cursorShape": "bar"` 
+ 光标颜色：`"cursorColor" : "#000000"`
+ 背景图片：`"backgroundImage" : "Image-Path/image.png"`

更多设置可以参考[官方文档](https://docs.microsoft.com/en-us/windows/terminal/customize-settings/profile-general) 。

## PowerShell 美化

在PowerShell美化中，我使用了`oh-my-posh`这个主题框架，它可以提供丰富的样式以供选择，感受还是相当不错的。

### 安装 oh-my-posh

> 此处需要注意：
>
> 如果此前没有安装 NuGet 提供程序，则安装时会提示安装 NuGet，输入Y安装即可。
>
> 如果安装时没有权限执行脚本，可能需要先执行 `Set-ExecutionPolicy`获取权限。
>
> > 建议安装前先执行`get-executionpolicy` 获取当前权限，若为`Restricted` (默认)，则推荐使用 `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` 命令，将权限设为 `RemoteSigned` 。


使用管理员模式开启Windows Terminal (PowerShell)，安装 `posh-git` 和 `oh-my-posh` 这两个模块。

```shell
Install-Module posh-git -Scope CurrentUser
Install-Module oh-my-posh -Scope CurrentUser
```

### 安装 Get-ChildItemColor

在图中可以看到，使用 `ls` 命令可以获得一个类似Linux的彩色输出，如何在Windows下实现这种好看的文件输出呢？我们可以安装 `Get-ChildItemColor` 模块。

执行以下命令安装：

```shell
Install-Module -AllowClobber Get-ChildItemColor
```

详细配置可以参考其[README](https://github.com/joonro/Get-ChildItemColor) 。

### 配置主题配置文件

通过以下命令测试，如果之前没有配置文件，就新建一个 PowerShell 配置文件。

```shell
if (!(Test-Path -Path $PROFILE )) { New-Item -Type File -Path $PROFILE -Force }
```

可以选择使用vscode或其他编辑器打开（选择其一）：

```shell
code $PROFILE
notepad $PROFILE
```

在其中添加：

```shell
Import-Module posh-git
Import-Module oh-my-posh
Set-PoshPrompt -Theme aliens
If (-Not (Test-Path Variable:PSise)) {
    Import-Module Get-ChildItemColor
    Set-Alias l Get-ChildItem -option AllScope
    Set-Alias ls Get-ChildItemColorFormatWide -option AllScope
}
```

其中1，2行导入刚才安装的 `oh-my-posh` 模块；第3行设置 `oh-my-posh` 的主题；最后几行设置 `Get-ChildItemColor` 模块，使 `l` 、`ls` 命令成为调用该模块命令的别名，以覆盖原有命令。

关于 `oh-my-posh` 的主题，官方目前给出了19款，可以前往其 [文档](https://ohmyposh.dev/docs/themes) 查看并选择自己喜欢的主题，这里个人推荐使用 `aliens` ，会比较适合浅色背景使用。

### 字体设置

由于`oh-my-posh` 的很多主题使用powerline字体，所以若没有设置字体为powerline字体，PowerShell中将出现很多方框。

在[这里](https://github.com/powerline/fonts)可以下载powerline字体，个人使用的是 `Meslo` 。

在Windows Terminal中应用该字体的方法可以参考上文："Windows Terminal 美化"中 [其他设置](#其他设置) 。

同时，也可以将vscode的终端字体设置为该字体:

1. 打开vscode的文件->首选项->设置 (ctrl+,)
2. 找到 Terminal > Integrated > Font Family 
3. 设置为该字体即可。

## 结语

至此，Windows Terminal的美化就大致完成了，好看才是第一生产力，毕竟心情愉悦才能提高效率不是？

