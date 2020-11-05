---
title: "本地构建支持多架构的Docker镜像"
date: 2020-11-01T14:37:08+08:00
showDate: true
showTags: true
showMeta: true
showSocial: true
showActions: true
showPagination: true
categories:
- docker
- image
tags:
- docker
- dockerfile
- multi-arch
keywords:
- Docker
autoThumbnailImage: yes
thumbnailImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/docker_750x236.png
thumbnailImagePosition: top
metaAlignment: center
coverImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/docker_1920x1080.jpeg
coverCaption: "集装箱？容器？Docker！"
coverMeta: in
coverSize: full
---

在个人电脑构建支持amd64、arm和arm64的多架构镜像

<!--more-->

而不是借助于Travis CI或者DockerHub，适用于内网环境

<!-- toc -->

# 用到的工具

- [buildx](https://github.com/docker/buildx)是一个Docker CLI插件，通过BuildKit扩展了构建功能
- [BuildKit](https://github.com/moby/buildkit)是一个并发、高效缓存且与DOckerfile无关的构建器工具箱
- [docker/binfmt](https://hub.docker.com/r/docker/binfmt)提供qemu模拟器来模拟不同架构

# 安装

buildx现在(2020-09-04)还是一个预览功能，默认只支持19.03或更高版本的Docker，有以下三种方式安装buildx：

## 直接启用

buildx从Docker 19.03开始与Docker CE集成到一起，但想要使用它还要在Docker CLI上启用实验模式

启用实验模式有以下两种方法，任选其一：

- 编辑`~/.docker/config.json`，添加如下内容：

    ```json
    {
        "experimental": "enabled"
    }
    ```

- 设置环境变量`DOCKER_CLI_EXPERIMENTAL=enabled`

## 二进制文件

直接安装buildx二进制文件（这能为旧版Docker提供**有限的功能集**）

1. 从[buildx - releases页面](https://github.com/docker/buildx/releases/latest)下载最新的二进制文件

2. 将该文件重命名为`docker-buildx`并复制到`~/.docker/cli-plugins/`下

3. 然后赋予其可执行权限：

    ```
    chmod x ~/.docker/cli-plugins/docker-buildx
    ```

## 从源码编译

也可以自己编译buildx，具体请参考[buildx#building](https://github.com/docker/buildx#building)

# 验证

使用以下命令验证buildx是否已经可用：

```bash
docker buildx version
```

# 构建

buildx可用之后，严格按照以下步骤构建多架构镜像

## 开启binfmt_misc

binfmt_misc用来支持多架构

如果使用的是Linux需要设置binfmt_misc，如果使用的是Windows或者OSX的Docker桌面版则不需要，因为Docker本来就需要binfmt_misc支持才能在这两个系统上运行

> 因为Docker是原生运行于Linux上的，而在Windows/OSX上是通过模拟一个Linux内核来运行的

1. 开启binfmt_misc

    在开启binfmt_misc之前执行以下命令：

    ```bash
    ll /proc/sys/fs/binfmt_misc
    ```

    可以看到该路径下只有'registry'和'status'两个文件：

    ![Before](https://gitee.com/YJ1516/MyPic/raw/master/picgo/before_binnfmt.png)

    接着执行以下命令开启binfmt_misc：

    ```bash
    docker run --privileged --rm tonistiigi/binfmt --install all
    ```

    再次查看`/proc/sys/fs/binfmt_misc`：

    ![After](https://gitee.com/YJ1516/MyPic/raw/master/picgo/after_binfmt.png)

2. 查看指定架构的解释器是否已启用

    查看`/proc/sys/fs/binfmt_misc`下新增的以'qemu'开头的文件内容：

    ```bash
    cat /proc/sys/fs/binfmt_misc/qemu-*
    ```

    ![QEMU interpreter](https://gitee.com/YJ1516/MyPic/raw/master/picgo/interpreter.png)

    可以看到每个文件的第一行都是'enabled'，即代表该架构处理程序已启用

    注意'interpreter'行的内容，这是要用到的程序，如果没有这些程序的话需要安装

    > Arch Linux中是qemu-arch-extra包（或qemu-headless-arch-extra，取决于依赖的是qemu还是qemu-headless）
    >
    > Ubuntu中是qemu-user包

## 切换默认的构建器

执行命令`docker buildx ls`可以看到Docker默认的构建器default只支持amd64和i386架构，所以要创建一个新的支持多架构的构建器：

1. 创建多架构构建器muilt-arch：

    ```bash
    docker buildx create --name muilt-arch
    ```

2. 将默认构建器切换为muilt-arch：

    ```bash
    docker buildx use muilt-arch
    ```

3. 检查当前构建器：

    ```bash
    docker buildx inspect muilt-arch --bootstrap
    ```

    > `--bootstrap`参数确保构建器启动

    ![inspect mailt-arch](https://gitee.com/YJ1516/MyPic/raw/master/picgo/inspect_mailt-arch.png)

    **这会拉取moby/buildkit镜像，确保网络连接正常或提前已完成拉取**

## 开始构建

接下来使用muilt-arch构建器进行构建即可

```bash
docker buildx build --file ./Dockerfile --platform linux/arm,linux/arm64,linux/amd64 --tag image-name:image-tag .
```

> 如果构建过程中报错：`failed to solve: rpc error: code = Unknown desc = failed to load LLB: runtime execution on platform linux/arm/v7 not supported`，尝试重新创建构建器

# 总结

使用多架构构建器来构建image需要满足以下条件：

1. 启用了buildx

2. 开启了binfmt_misc

    必须运行`docker run --privileged --rm tonistiigi/binfmt --install all`

3. 确保使用的是多架构构建器

    必须运行`docker buildx use muilt-arch`以启用多架构构建器

    **确保星号在muilt-arch上并且其Status是running**：

    ![builder](https://gitee.com/YJ1516/MyPic/raw/master/picgo/builder-2.png)

4. 构建时指定了架构

# 参考

[Getting started with Docker for Arm on Linux](https://www.docker.com/blog/getting-started-with-docker-for-arm-on-linux/)
