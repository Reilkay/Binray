---
title: 使用Github Actions自动部署Hugo到Github Pages
date: 2020-07-17 02:10:08
updated: 2020-07-17 02:10:08
tags: ["Hugo","博客"]
comments: true
---

前段时间看到好多人用Github Actions搞自动化，最近又有Dalao告诉我能用Action自动部署博客，顿时有些心痒，毕竟Hugo博客每次更新的时候要删掉public里的内容重新生成，一不小心就会把git文件也删掉（好像暴露了不经常写博客所以不熟练的事实），遂上网查了查，没什么中文的文档，于是在此记录一下折腾的经历，以备不时之需。

<!-- more -->

## 建立分支上传源文件

首先需要先建一个新的分支把整个博客的源文件传上去，因为我是网页已经在master分支上挂着了不想折腾，所以又建立了source分支用于放置源文件（记得把源文件的分支设为default）。

![img](https://i.loli.net/2020/07/17/KPXEr1noR8DqwSv.png)

注：事实上可以用master分支放置源文件，而新建一个gh-page分支挂网页，这样可以减少一些不必要的操作~

## 在Actions中新建workflow

1. 首先打开Actions页面，点击新建workflow。

    ![img](https://i.loli.net/2020/07/17/9ZrvTxWMlX8pdQf.png)

    ![img](https://i.loli.net/2020/07/17/QGb4jmJesqyrXta.png)

2. 之后我们使用现成的项目:

    [![image-20200717142237607](https://i.loli.net/2020/07/17/IGAUmnKEV73eh8b.png)](https://github.com/peaceiris/actions-hugo)

    根据README中给出的代码填写。

    ![image-20200717142456613](https://i.loli.net/2020/07/17/F9exYaksP7TNbrz.png)

    此时注意：如果源代码在master分支，则不需要进行下一步。
    
3. 修改第六行`- master`为你的源代码所在的分支，并在末尾添加一行`publish_branch: master`。

    例如我这里：

    ![img](https://i.loli.net/2020/07/17/bIZhDCUzBGKqcay.png)

4. Commit到源码分支即可

   ![img](https://i.loli.net/2020/07/17/LbaptGHQTyNrc5P.png)

## 使用

无需进行其他操作，在每次更新完博客后，将源文件commit至源码所在分支，即可实现自动生成html并部署至GitHub Pages。