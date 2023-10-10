---
title: "OverlayFs"
date: 2019-10-18T09:54:13+08:00
showDate: true
showTags: true
showMeta: true
showSocial: true
showActions: true
showPagination: true
categories:
- linux
- filesystem
tags:
- overlayfs
- linux
- docker
- 文件系统
- filesystem
keywords:
- linux
- fs
- filesystem
- overlay
- 文件系统
- union
- docker
- 文件系统
autoThumbnailImage: yes
thumbnailImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/Qmccree.jpg
thumbnailImagePosition: left
metaAlignment: center
coverImage: https://i.loli.net/2019/10/18/XucUboBxtyQjfNT.jpg
coverCaption: "Heaven seals off all exits"
coverMeta: in
coverSize: full
---

OverlayFs(Overlay Filesystem)是Linux的一种文件系统——**{{< hl-text orange >}}union mount filesystem（联合挂载文件系统）{{< /hl-text >}}**，自Linux Kernal 3.18开始合并到linux内核中

<!--more-->

<!-- toc -->

Docker使用overlay 2作为存储驱动：

{{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/docker-overlayfs.png" thumbnail="" group="group:overlay" thumbnail-width="100%" thumbnail-height="100%" title="OverlayFS" >}}


# 概述

OverlayFs将一个**read-write**目录树覆盖到另一个**read-only**目录树上，对文件的所以修改都在**read-write**的上层(*{{< hl-text primary >}}layer{{< /hl-text >}}*)进行，这类机制常用于live CD、docker等，但还有许多其他用途

---

# 安装

在3.18以上版本的Linux内核主线(Kernal mainline)中，OverlayFs默认是启用的，当有对OverlayFs的`mount`需求时，`overlay`模块会自动加载，可以使用`lsmod | grep overlay`命令查看，只要输出里有`overlay`关键字代表模块已加载

{{< alert info >}}当然，在其他个人或组织释出的第三方（区别于Linus释出的）Linux内核中并不确定`OverlayFs`是否默认启用{{< /alert >}}

---

# 使用

要挂载一个overlay，使用`mount`命令，参数如下：

{{< codeblock "bash" "bash" >}}
mount -t overlay overlay -o lowerdir=/path/to/lower1:/path/to/lower2,upperdir=/path/to/upperdir,workdir=/path/to/workdir /path/to/mergedir
{{< /codeblock >}}

{{< alert info >}}lowerdir可以由多个文件夹组成，以 **:** 分隔{{< /alert >}}

## 具体操作

1. 首先创建4个文件夹

    {{< codeblock "bash" "bash" >}}
    mkdir lower upper work merged   # 底层目录 上层目录 工作目录 挂载目录
    {{< /codeblock >}}

2. 然后在这次个文件夹的统计目录使用`mount`命令进行挂载

    {{< codeblock "bash" "bash" >}}
    mount -t overlay overlay -o lowerdir=./lower,upperdir=./upper,workdir=./work ./merged
    {{< /codeblock >}}

    {{< alert warning >}}**workdir**必须是一个和**upperdir**位于同一个filesystem上的空文件夹{{< /alert >}}

3. 使用`df -h`命令查看挂载结果，结果如下：

    {{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/overlay_df.png" thumbnail="" group="group:overlay" thumbnail-width="100%" thumbnail-height="100%" title="df -h" >}}

    可以看到已经有一个overlay文件系统了，该文件系统的挂载点是'merged'的路径；"容量"显示的是**upperdir**的信息

4. 写入数据

    - 对**merged**中的数据的所有修改都将反映到**upper**：

        {{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/merged.png" thumbnail="" group="group:overlay" thumbnail-width="100%" thumbnail-height="100%" title="merged" >}}

        {{< alert info >}}从上图可以看到，一开始**lowerX**(X=1/2/3)、**upper**、**merged**是空文件夹，在**merged**里创建了一个文件**mergedfile**，内容是"touch <mergedfile> file in merged"，并没有对**upper**进行操作，但还是可以看到**upper**里也出现了一个**mergedfile**文件，并且内容也和**merged/mergedfile**相同{{< /alert >}}

    - 对**upper**中的数据的所有修改都将反映到**merged**：

        {{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/upper.png" thumbnail="" group="group:overlay" thumbnail-width="100%" thumbnail-height="100%" title="upper" >}}

        {{< alert info >}}从上图可以看到，在**upper**里新建了一个文件**upperfile**，内容是"touch <upperfile> file in upper"，**merged**里自动出现了**upperfile**文件，内容和**upper/upperfile**一样{{< /alert >}}

---

# Read-only overlay

当**upperdir**和**workdir**参数未给出的时候，overlay挂载为只读

{{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/readonly.png" thumbnail="" group="group:overlay" thumbnail-width="100%" thumbnail-height="100%" title="read only" >}}

{{< alert warning>}}挂载只读overlay时候需要最少两个**lowerdir**{{< /alert >}}

---

# 特性

- 只有**merged**对用户是可见的，用户不能直接操作**upperdir**和**lowerdir**
- 对**merged**的所有操作都反映到**upperdir**，但不会反映到**lowerdir**，overlay挂载以后，**lowerlay**对其来说就是无关的了（怀疑是overlay挂载时拷贝了**lowerdir**），这对**lowerdir**的数据起到了保护作用，docker中对container的修改不会影响到image就是因为这个特性
- **loderdir**和**upperdir**之间并没有数据交互，即对一方的修改并不会反映到另一方
- 当**lowerdir**和**upperdir**中有同名文件的时候，在挂载overlay时会把**upperdir**中的该文件反映到**merged**，**lowerdir**独有的文件也会反映到**merged**，因此在运行一个docker容器的时候可以使用image里默认的参数，也可以在运行时修改参数以覆盖默认值，其他没有被修改的参数还是使用image里的默认值

---

关于OverlayFs的详细信息请看：[Overlay Filesystem documentation](https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt)
