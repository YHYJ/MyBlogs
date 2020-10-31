---
title: "Git LFS的使用方法"
date: 2020-10-31T23:43:54+08:00
showDate: true
showTags: true
showMeta: true
showSocial: true
showActions: true
showPagination: true
categories:
- Git
- LFS
tags:
- git
- large file
keywords:
- git
autoThumbnailImage: yes
thumbnailImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/QdeathQ.jpg
thumbnailImagePosition: left
metaAlignment: center
coverImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/sky_1920x1080.jpg
coverCaption: "Fly to the sky"
coverMeta: in
coverSize: full
---

LFS是一个Git扩展，在对大文件进行版本控制的同时保持仓库的体积不至于太大

Github仓库限制文件大小不能超过100MB，因此在包含大文件的仓库中使用LFS是很有必要的

<!--more-->

<!-- toc -->

# 安装

Linux：一般是包含在软件源里了的，包名是`git-lfs`，使用包管理器进行安装即可

Windows：可以到[下载链接](https://github.com/git-lfs/git-lfs/releases)下载安装包进行安装

MAC：可以使用`Homebrew`进行安装

# 使用

安装完成之后可以使用`git-lfs`或者`git lfs`命令管理大文件，本质是一样的

1. 设置LFS管理该仓库

    ```bash
    git lfs install
    ```

    该命令只能在git仓库里运行，执行之后即可使用lFS管理该仓库的大文件

2. 追踪大文件

    ```bash
    git lfs track README.md
    ```

    执行该命令以追踪指定文件（示例中追踪README.md），或者可以使用通配符：

    ```bash
    git lfs track *.md
    ```

    该命令会生成一个名为`.gitattributes`，文件内容格式如图：

    ![.gitattributes](https://gitee.com/YJ1516/MyPic/raw/master/picgo/20201101_014414.png)

    也可以不带文件名执行该命令，这样可以查看当前的追踪模式：

    ![git lfs track](https://gitee.com/YJ1516/MyPic/raw/master/picgo/20201101_014701.png)

3. 推送.gitattributes

    **注意，大文件追踪名单完成后需要先将`.gitattributes`文件push到位于github的云端仓库，之后添加大文件才不会有100MB的限制提示**

    添加了LFS支持的仓库除了针对于LFS的操作之外其他操作和普通仓库没有区别，例如推送到远端：

    ```bash
    git push
    ```

    现在可以将大文件添加到仓库了

4. 查看当前跟踪的文件

    .gitattributes文件提交后可以运行以下命令查看当前跟踪的文件列表：

    ```bash
    git lfs lf-files
    ```

    ![ls files](https://gitee.com/YJ1516/MyPic/raw/master/picgo/20201101_022958.png)
