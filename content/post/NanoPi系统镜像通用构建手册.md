h NanoPi系统镜像通用构建手册

本文只介绍FriendlyCore（基于Ubuntu Core定制）系统的定制安装方式，不涉及到FriendlyWrt（基于OpenWrt定制）系统

其中系统安装方式有2种：

- 安装到TF卡

    NanoPi有外置的TF卡槽，可以自由选配合适规格的TF使用，并且可以将TF卡插到电脑上批量烧写系统

- 安装到eMMC

    部分NanoPi主板上焊接了eMMC（例如NanoPi-R1有8GB板载eMMC），eMMC容量是固定的不可自由选配，但因为是直接焊接到主板上的，有稳定、高速等优点

    而且因为有板载eMMC的NanoPi一般出厂时在eMMC预装了FriendlyWrt，所以可以直接开始使用，即使也有办法将eMMC格式化重新安装系统

---

## Table of Contents

<!-- vim-markdown-toc GFM -->

* [重要提示](#重要提示)
* [FriendlyCore](#friendlycore)
    * [1. 前期准备](#1-前期准备)
        * [1.1 CPU型号](#11-cpu型号)
        * [1.2 构建工具包](#12-构建工具包)
            * [编译工具链](#编译工具链)
            * [NanoPi-R1 (H3)](#nanopi-r1-h3)
            * [NanoPi-R2S (RK3328)](#nanopi-r2s-rk3328)
        * [1.3 系统文件](#13-系统文件)
            * [NanoPi-R1 (H3)](#nanopi-r1-h3-1)
            * [NanoPi-R2S (RK3328)](#nanopi-r2s-rk3328-1)
        * [1.4 烧写工具](#14-烧写工具)
        * [1.5 需要的系统](#15-需要的系统)
    * [2. 目录结构](#2-目录结构)
        * [2.1 压缩文件](#21-压缩文件)
        * [2.2 解压文件](#22-解压文件)
    * [3. 构建系统](#3-构建系统)
        * [3.1 进入chroot环境](#31-进入chroot环境)
        * [3.2 修改rootfs](#32-修改rootfs)
        * [3.3 退出chroot环境](#33-退出chroot环境)
        * [3.4 将修改后的rootfs打包到我的电脑](#34-将修改后的rootfs打包到我的电脑)
        * [3.5 构建rootfs.img](#35-构建rootfsimg)
            * [NanoPi-R1](#nanopi-r1)
            * [NanoPi-R2S](#nanopi-r2s)
        * [3.6 构建可启动TF卡](#36-构建可启动tf卡)
        * [3.7 构建TF卡系统镜像](#37-构建tf卡系统镜像)
        * [3.8 构建eMMC系统镜像](#38-构建emmc系统镜像)
        * [3.9 后记说明](#39-后记说明)
    * [4 系统烧写](#4-系统烧写)
        * [4.1 Linux系统下的烧写步骤](#41-linux系统下的烧写步骤)
        * [4.2 Windows系统下的烧写步骤](#42-windows系统下的烧写步骤)

<!-- vim-markdown-toc -->

---

确定系统型号 --> 下载构件工具包 --> 下载系统文件 --> 调整目录结构 --> 得到所有系统镜像 --> 修改rootfs.img --> 执行构建脚本

参考链接：

- [NanoPi-R1 Wiki](http://wiki.friendlyarm.com/wiki/index.php/NanoPi_R1/zh)
- [NanoPi-R2S Wiki](http://wiki.friendlyarm.com/wiki/index.php/NanoPi_R2S/zh)
- [NanoPi-M1 Wiki（包含“如何编译Linux系统”）](http://wiki.friendlyarm.com/wiki/index.php/NanoPi_M1/zh)
- [Building U-boot and Linux for H5/H3/H2+](http://wiki.friendlyarm.com/wiki/index.php/Building_U-boot_and_Linux_for_H5/H3/H2%2B/zh)
- [Google云盘_H3_NanoPi-R1](https://drive.google.com/drive/folders/14CMDF5OvlvdeuHezPhwZJTSuzi_VlwpR)
- [EFlasher](http://wiki.friendlyarm.com/wiki/index.php/EFlasher/zh)
- [EFlasher-updater](https://github.com/friendlyarm/eflasher-updater)
- [官方存储服务器](http://112.124.9.243)

---

## 重要提示

- ***文档中涉及到系统构建的命令最好都由root用户（不是root权限）执行，否则可能会导致烧写好的系统中的命令出现权限错误***

    当然文件上传下载等命令并不需要

- ***文档中进入chroot环境之后，只要没有明确说退出chroot环境，在这之间所有的操作都是在chroot环境中进行的***

## FriendlyCore

### 1. 前期准备

#### 1.1 CPU型号

首先需要看一下CPU确定CPU型号：

> 找不到CPU图片或者官方没给出CPU型号的，拆开外壳直接看一下CPU即可

![NanoPi-R1接口布局图](https://gitee.com/YJ1516/MyPic/raw/master/picgo/NanoPi_R1-layout.jpg)

由以上NanoPi-R1接口布局图可以看出R1使用的是[全志(Allwinner) H3](https://linux-sunxi.org/H3)系列CPU

![NanoPi-R2S接口布局图](https://gitee.com/YJ1516/MyPic/raw/master/picgo/NanoPi_R2S-layout.jpg)

由以上NanoPi-R2S接口布局图可以看出R2S使用的是[瑞芯(Rockchip) RK3328](http://opensource.rock-chips.com/wiki_RK3328)系列CPU

#### 1.2 构建工具包

CPU型号确定之后到[官方仓库](https://github.com/friendlyarm?tab=repositories)下载对应的构件工具包（用于构建ROM文件）

##### 编译工具链

在最后[构建系统镜像](#36-构建系统镜像)的时候需要用到[编译工具链](https://github.com/friendlyarm/prebuilts)，安装方法参照其README文档

##### NanoPi-R1 (H3)

NanoPi-R1（或者说H3）需要以下仓库：

- [构件工具包](https://github.com/friendlyarm/sd-fuse_h3)

    **重要**，用于构建可启动TF卡/ROM文件

##### NanoPi-R2S (RK3328)

NanoPi-R2S（或者说RK3328）需要以下仓库：

- [构件工具包](https://github.com/friendlyarm/sd-fuse_rk3328.git)

    **重要**，用于构建可启动TF卡/ROM文件

#### 1.3 系统文件

CPU型号确定之后到[官方存储服务器](http://112.124.9.243/dvdfiles/)找到对应的系统文件存储位置（作为构建系统的“原料”）

> [H3系列CPU系统文件存储在这](http://112.124.9.243/dvdfiles/H3/images-for-eflasher)
>
> [RK3328系列CPU系统文件存储在这](http://112.124.9.243/dvdfiles/RK3328/images-for-eflasher)

##### NanoPi-R1 (H3)

NanoPi-R1（或者说H3）需要以下系统文件：

- [friendlycore-zenial_4.14_armhf](http://112.124.9.243/dvdfiles/H3/images-for-eflasher/friendlycore-xenial_4.14_armhf.tgz)

    **重要**，包含所有系统文件

- [rootfs_friendlycore_4.14](http://112.124.9.243/dvdfiles/H3/rootfs/rootfs_friendlycore_4.14.tgz)

    **重要**，用于构建rootfs.img

- [eflasher](http://112.124.9.243/dvdfiles/H3/images-for-eflasher/eflasher.tgz)

    **重要**，用于构建eflasher固件（该固件用于通过TF卡将ROM烧写到eMMC）

##### NanoPi-R2S (RK3328)

NanoPi-R2S（或者说RK3328）需要以下系统文件：

- [friendlycore-arm64-images](http://112.124.9.243/dvdfiles/RK3328/images-for-eflasher/friendlycore-arm64-images.tgz)

    **重要**，包含所有系统文件

- [rootfs-friendlycore-arm64](http://112.124.9.243/dvdfiles/RK3328/rootfs/rootfs-friendlycore-arm64.tgz)

    **重要**，用于构建rootfs.img

> 因为NanoPi-R2S没有eMMC，这里不讨论它的eflasher烧写方法

#### 1.4 烧写工具

[下载地址](http://download.friendlyarm.com/nanopir2s)

Windows下将系统镜像烧写到NanoPi的软件（Linux使用`dd`命令即可），给出了百度网盘和Google云盘下载地址，看情况选择其中一个

下载'tools/win32diskimager.rar'，该工具对所有NanoPi是通用的

#### 1.5 需要的系统

整个构建流程需要在以下系统中进行：

> 系统架构用`uname -m`命令查询

- 一个已安装官方friendlyCore系统的NanoPi-R1

    为了方便，之后提到"**NanoPi-R1系统**"即指代该系统

    NanoPi-R1系统用来对rootfs.img进行修改

    > NanoPi-R1的CPU硬件架构是armv7l

- 一个x86_64架构的Linux系统

    实操用的是个人电脑

    为了方便，之后提到"**我的电脑**"即指代该系统

    我的电脑用来运行构建脚本

### 2. 目录结构

[1. 前期准备]((#1-前期准备)完成后，需要确保文件符合目录结构：

#### 2.1 压缩文件

- NanoPi-R1压缩文件应符合以下目录结构：

    ```
    build_image
    ├── images-for-eflasher
    │   ├── eflasher.tgz
    │   └── friendlycore-xenial_4.14_armhf.tgz
    ├── rootfs
    │   └── rootfs_friendlycore_4.14.tgz
    └── sd-fuse_h3.tgz
    ```

- NanoPi-R2S压缩文件应符合以下目录结构：

    ```
    build_image
    ├── images-for-eflasher
    │   └── friendlycore-arm64-images.tgz
    ├── rootfs
    │   └── rootfs-friendlycore-arm64.tgz
    └── sd-fuse_rk3328.tgz
    ```

#### 2.2 解压文件

> 这里展示的是之后步骤里解压缩2.1中的压缩包之后的文件结构，不代表当前结构
>
> 如果发现自己没有2.2中列出的文件不用在意，继续执行后面步骤就有了

- NanoPi-R1压缩文件解压后应符合以下目录结构：

    ```
    sd-fuse_h3
    ├── eflasher
    └── friendlycore-xenial_4.14_armhf
        └── rootfs
    ```

- NanoPi-R2S压缩文件解压后应符合以下目录结构：

    ```
    sd-fuse_rk3328
    └── friendlycore-xenial_4.14_armhf
        └── rootfs
    ```

### 3. 构建系统

[1. 前期准备](#1-前期准备)完成且[2. 目录结构](#2-目录结构)没问题之后，开始进行系统构建

> 以下提到的进到某个路径或者打开某个文件的命令/描述，可能会因为R1和R2S的路径/文件名不同而不同，但大体上是相同的（不同之处以星号(*)代替），理应能够自行区分

#### 3.1 进入chroot环境

***该部分全部操作都需要在armv7l架构的Linux系统中进行，即[1.5 需要的系统](#15-需要的系统)中所说的NanoPi-R1系统***

> 使用chroot以rootfs作为当前进程及其子进程的根目录，以此来修改rootfs，这种环境叫做'chroot jail'

1. 将`/Path/to/build_image/rootfs/rootfs*friendlycore*.tgz`文件上传到NanoPi-R1系统

    可以使用FileZilla软件或者`scp`命令

2. 进到`/Path/to/rootfs*friendlycore*.tgz`文件路径

3. 解压缩rootfs*friendlycore*.tgz文件

    ```bash
    bsdtar -xzvpf rootfs*friendlycore*.tgz
    ```

    > 使用`bsdtar`而非`tar`命令解压缩文件，`tar`解压缩最后会报错：*Directory renamed before its status could be extracted*，该错误可能是overlay文件系统导致的
    >
    > 绝对不能漏掉`-p`参数，否则会导致文件属性错误，并且`-p`和`-f`的顺序不能错

    R1和R2S解压缩得到的文件夹不同：

    - R1直接得到rootfs文件夹
    - R2S得到friendlycore-arm64/rootfs

    但只是R2S比R1多一层friendlycore-arm64目录而已，rootfs文件夹下的内容都是类似的，以下以`/Path/to/rootfs`代指该路径

    解压得到的`/Path/to/rootfs`中有以下内容：

    ![rootfs list](https://gitee.com/YJ1516/MyPic/raw/master/picgo/rootfs_ll.png)

4. 挂载临时API文件系统

    ```bash
    mount -t proc proc /Path/to/rootfs/proc/
    mount -o bind /sys /Path/to/rootfs/sys/
    mount -o bind /dev /Path/to/rootfs/dev/
    mount -o bind /run /Path/to/rootfs/run/
    ```

    尽量使用`-o bind`选项而非`--rbind`，`--rbind`会导致'dev/'和'sys/'的某些子目录无法卸载

5. 建立网络连接

    如果NanoPi-R1系统已经建立了一个网络连接并且想在chroot jail中继续使用该连接，需要将NanoPi-R1系统的DNS服务器配置文件复制到chroot jail：

    ```bash
    cp /etc/resolv.conf /Path/to/rootfs/etc/resolv.conf
    ```

    > 如果提示：cp: '/etc/resolv.conf' and 'friendlycore-arm64/rootfs/etc/resolv.conf' are the same file，忽略即可

6. 进入chroot环境

    ```bash
    chroot /Path/to/rootfs /bin/bash
    ```

    以`/Path/to/rootfs`作为运行`/bin/bash`命令时的根目录

***以下[3.2 修改rootfs](#32-修改rootfs)所有命令都在chroot环境中执行，直到[3.3 退出chroot环境](#33-退出chroot环境)为止***

#### 3.2 修改rootfs

***该部分全部操作都需要在armv7l架构的Linux系统中进行，即[1.5 需要的系统](#15-需要的系统)中所说的NanoPi-R1系统***

***注意要使用root用户（不是root权限）执行命令，否则可能会导致烧写好的系统中的命令出现权限错误***

按照[3.1 进入chroot环境](#31-进入chroot环境)步骤成功进入chroot环境，在退出之前所做的任何操作都以`/Path/to/rootfs`为根目录`/`进行，不会影响到chroot之外

1. 设置时区

    使用上海时区：

    ```bash
    dpkg-reconfigure tzdata
    ```

    依次选择**Asia** - **Shanghai**

    > 检查systemd-timesyncd.service的状态，应为inactive，因为systemd-timesyncd检测到了ntpd的存在，认为当前时间同步由ntpd完成
    >
    > 再检查ntp.service的状态，是active，时间同步确实由ntp完成

2. 配置locale

    默认locale为`en_US.UTF-8`，再加上中文的，执行以下命令修改：

    ```bash
    dpkg-reconfigure locales
    ```

    该命令会提供一个TUI的对话框供选择locale（使用“↑”，“↓” 键定位到选项之后使用空格键选定），选择以下项：

    ```bash
    en_US.UTF-8 UTF-8
    zh_CN.GBK GBK
    zh_CN.UTF-8 UTF-8
    ```

    全部选中之后按Tab键跳到<OK>上，按Enter键，之后还会询问想要设置的默认locale，选择`en_US.UTF-8`即可

    之后会生成locales缓存，稍等一会儿

3. 修改HOME目录默认文件

    原始的`ll`命令（其实是个alias）默认查看所有文件（包括隐藏文件），对于日常使用来说不方便

    alias由HOME目录下的.bashrc文件定义，修改新建用户时的默认.bashrc以改变默认的`ll`的行为

    新增用户的默认文件由`/etc/skel`定义，该文件夹下的所有文件将会复制到新用户的HOME路径下，因此修改`/etc/skel/.bashrc`即可

    编辑`/etc/skel/.bashrc`，找到其中的'alias'行，将下图中的三行alias删除：

    ![alias](https://gitee.com/YJ1516/MyPic/raw/master/picgo/alias.png)

    新增以下内容：

    ```bash
    alias ll='ls -lhiFgHS'
    alias lla='ls -AlhiFgHS'
    ```

    > `ll`显示所有非隐藏文件，`lla`显示包括隐藏文件在内的所有文件

    *顺便可以修改`/root/.bashrc`*

4. 配置软件源

    > 默认软件源是Ubuntu官方源，服务器位于国外，网络状况很差，修改`/etc/apt/sources.list`问将，将其替换为国内镜像源

    将[cn-apt-source.sh](./script/cn-apt-source.sh)文件复制到`/Path/to/rootfs/root`下（root用户的HOME目录），然后执行以下命令：

    ```bash
    bash /path/to/cn-apt-source.sh
    ```

    这将会备份原有的sources.list，然后写入国内镜像源

    可选的镜像源有清华和中科大，修改cn-apt-source.sh中的`APTSERVER`变量即可

5. 更新系统

    ```bash
    apt update
    apt upgrade
    ```

    > 过程中会询问要使用新版本的包里的issue配置文件还是保留旧版，默认是保留旧版(N)，建议使用新版(Y)

6. 搭建环境

    - 通过源安装软件包，执行以下命令：

        ```bash
        apt install tree bsdtar docker.io docker-compose freetds-dev
        ```

    - 其他软件包安装和环境配置参考[环境搭建手册.md](./docs/环境搭建手册.md)

    - Python安装完成之后执行以下命令装包：

        ```bash
        pip3 install wheel cython redis toml influxdb pymodbus==1.4.0 python-etcd python-snap7 pendulum falcon msgpack-python waitress pymssql-linux logbook pykafka kafka-python pyyaml
        ```

        > 其中：
        >
        > 将`pip3`命令替换为实际的Python版本对应的`pip`命令
        >
        > pymssql-linux需要先通过apt安装freetds-dev

        之后将Doctopus源码包导入到/Path/to/rootfs，解压缩之后使用其中的setup.py安装：

        ```bash
        python3 setup.py install
        ```

        > 其中：
        >
        > 将`python3`命令替换为实际的Python版本对应的`python`命令

    - 安装libsnap7

        1. 将[libsnap7.so](./source/libsnap7.so)复制到`/usr/local/lib/snap7`（没有该路径可以手动创建）

        2. 创建`/etc/ld.so.conf.d/snap7.conf`文件，写入以下内容：

            ```bash
            /usr/local/lib/snap7
            ```

        3. 执行以下命令以加载lisnap7.so：

            ```bash
            ldconfig
            ```

7. 配置网络

    NanoPi-R1有三个网卡：两个有线网卡（eth0、eth1）和一个无线网卡（wlan0）

    NanoPi-R2S有两个有线网卡，没有无线网卡

    > NanoPi-R1官方系统默认安装了network-manager和ifupdown来管理网络；NanoPi-R2S官方系统默认安装了ifupdown来管理网络
    >
    > network-manager提供实用工具`nmcli`和网络管理服务`NetworkManager.server`
    >
    > ifupdown提供实用工具`ifup`、`ifdown`和网络管理服务`networking.service`

    *因为NanoPi-R2S网络情况相对“单纯”，以下只讨论NanoPi-R1的网络配置，NanoPi-R2S参照NanoPi-R1的有线网卡部分进行配置即可*

    系统默认使用`NetworkManager.service`管理所有网络，`networking.service`是不启动的，但因为network-manager配置静态IP不方便，所以改为使用ifupdown管理有线网络，network-manager管理无线网络

    1. 禁止network-manager管理有线网络

        编辑`/etc/NetworkManager/NetworkManager.conf`，确保其中'managed'项值为`false`：

        ![NetworkManager.conf](https://gitee.com/YJ1516/MyPic/raw/master/picgo/nm_conf.png)

        编辑完成后重启NetworkManager.service，执行`nmcli dev`命令应看到eth0、eth1网卡对于network-manager来说是"unmanaged"：

        ![nmcli dev](https://gitee.com/YJ1516/MyPic/raw/master/picgo/nmcli_dev.png)

    2. 配置有线网络

        > 示例网络情况为：eth0配置为192.168.10.10/24（用来ssh连接NanoPi），eth1配置为DHCP自动获取

        编辑eth0网络配置文件`/etc/network/interfaces`，**新增**内容如下：

        ```bash
        auto eth0
        iface eth0 inet static
        address 192.168.10.10
        netmask 255.255.255.0

        auto eth1
        iface eth1 inet dhcp
        ```

        如果需要添加网关(gateway)或DNS(dns-nameserves)，则在对应网卡配置项（例如eth0）下添加：

        ```bash
        iface eth0 inet static
        address 192.168.10.10
        netmask 255.255.255.0
        gateway 192.168.10.254
        dns-nameservers 8.8.8.8
        ```

        > 1. 正常情况下可以在`/etc/network/interfaces.d`下创建网络配置文件，`/etc/network/interfaces`会自动调用该配置文件，但是实测NanoPi-R1这样配置的话网络不通，NanoPi-R2S可以，因此统一在`/etc/network/interfaces`配置网络

    3. 配置networking.service

        networking.service默认有35秒的超时，导致系统开机缓慢，减小超时时间以加速系统启动

        编辑`/etc/systemd/system/networking.service.d/reduce-timeout.conf`，其中有：

        ```bash
        [Service]
        TimeoutStartSec=30
        TimeoutStopSec=5
        ```

        将'TimeoutStartSec'和'TimeoutStopSec'的值都改为1（**不能改为0**）：

        ```bash
        [Service]
        TimeoutStartSec=1
        TimeoutStopSec=1
        ```

    4. 配置无线网络

        无线网络**不预先配置**，根据实际网络情况使用以下命令即可连接无线网络：

        ```bash
        nmcli dev wifi                                          # 列出扫描到的wifi网络
        nmcli dev wifi rescan                                   # 重新扫描wifi网络
        nmcli dev wifi connect '<SSID>' password '<password>'   # 连接指定wifi，<SSID>为wifi名，<password>是wifi密码
        ```

8. 用户设置

    - 删除多余用户

        ```bash
        userdel -r <username>
        ```

    - 修改指定用户密码

        ```bash
        passwd <username>
        ```

    - 新增用户并设置密码

        ```bash
        useradd -m -G sudo,docker -s /bin/bash <username>      # 设置新增用户<username>的附加属组为sudo和docker，登陆shell为/bin/bash
        passwd <username>
        ```

        > 将新增用户添加到sudo组以赋予其sudo提权的能力，添加到docker组以赋予其不提权执行docker命令的权限

    - 将指定用户添加到指定组

        ```bash
        usermod -aG <group1>,<group2>,<group3> <username>
        ```

        > `-a`：将用户追加至`-G`参数列出的附加组中，并不从其它组中删除此用户
        >
        > `-G`：要将用户追加到的组，是一个列表

9. 系统清理

    - 清除孤立包（即未被其他软件包依赖也不依赖其他软件包）

        ```bash
        apt autoremove
        ```

    - 清除缓存文件

        ```bash
        rm /var/cache/apt/pkgcache.bin
        rm /var/cache/apt/srcpkgcache.bin
        ```

        这两个文件会在`apt update`时自动创建

        ```bash
        rm -r /var/lib/apt/lists/*
        ```

        该路径下是软件包列表本地缓存信息，可以安全删除，下次运行`apt update`会自动重建

        > 在再次运行`apt update`之前`apt-cache`之类的命令无法提供信息

        ```bash
        rm -r ~/.cache
        ```

    - 清除日志文件

        ```bash
        > /var/log/dpkg.log
        > /var/log/alternatives.log
        > /var/log/fontconfig.log
        > /var/log/apt/term.log
        > /var/log/apt/history.log
        ```

        **使用以上命令将这些文件内容清空而非`rm`删除它们，因为这可能会导致系统问题**

    - 清除命令历史记录

        ```bash
        rm ~/.nmcli-history
        rm ~/.python_history
        rm ~/.bash_history
        history -c
        ```

#### 3.3 退出chroot环境

rootfs修改完成后需要退出chroot环境，然后解除挂载

1. 退出chroot环境：

    使用Ctrl+d快捷键（`exit`命令退出会留下记录）

    退出chroot环境之后会回到`/Path/to/rootfs`路径下

2. 解除挂载：

    ```bash
    umount /Path/to/rootfs/proc
    umount /Path/to/rootfs/sys
    umount /Path/to/rootfs/dev
    umount /Path/to/rootfs/run
    ```

#### 3.4 将修改后的rootfs打包到我的电脑

***该部分全部操作都需要在armv7l架构的Linux系统中进行，即[1.5 需要的系统](#15-需要的系统)中所说的NanoPi-R1系统***

1. 打包压缩：

    ```bash
    bsdtar -czvpf rootfs-friendlycore-4.14_Mabo-$(date "+%Y%m%d")-NUM.tgz /Path/to/rootfs                    # NanoPi-R1的命名格式
    ```

    或

    ```bash
    bsdtar -czvpf rootfs-friendlycore-arm64_Mabo-$(date "+%Y%m%d")-NUM.tgz /Path/to/friendlycore-arm64       # NanoPi-R2S的命名格式
    ```

    > 其中'NUM'代表是当天第几次打包

    ***注意保存好该文件，之后可以在此基础上继续更新***

2. 传输到我的电脑：

    可以使用FileZilla软件或者`scp`命令

#### 3.5 构建rootfs.img

***该部分全部操作都需要在x86_64架构的Linux系统中进行，即[1.5 需要的系统](#15-需要的系统)中所说的我的电脑***

***注意要使用root用户（不是root权限）执行命令，否则可能会导致烧写好的系统中的命令出现权限错误***

> 该部分中：
>
> 1. 用到[构建工具包](#12-构建工具包)和[系统文件](#13-系统文件)
> 2. 命令涉及到相对路径的，请以`sd-fuse_*`目录为基准参考[2. 目录结构](#2-目录结构)
> 3. 用到的脚本的参数可以使用`bash <脚本名>`查看

##### NanoPi-R1

1. 进到NanoPi-R1的构建工具包sd-fuse_h3，将NanoPi-R1修改后的rootfs的压缩包rootfs_friendlycore_4.14*.tgz解压：

    ```bash
    bsdtar -xzvpf ../images-for-eflasher/friendlycore-xenial_4.14_armhf.tgz                      # 将friendlyCore完整系统文件解压，得到friendlycore-xenial_4.14_armhf文件夹
    bsdtar -xzvpf ../rootfs/rootfs-friendlycore-4.14*.tgz -C friendlycore-xenial_4.14_armhf      # 将rootfs解压到friendlycore-xenial_4.14_armhf/rootfs
    ```

    > 将*替换为具体文件名

2. 使用`build-rootfs-img.sh`脚本构建rootfs.img：

    > 脚本中用到`tempfile`命令创建临时目录，该命令只有Debian发行版才有（默认自带）
    >
    > Arch Linux发行版对应的命令为`mktemp`，编辑build-rootfs-img.sh文件将第41行`TMPFILE`变量的值改为`mktemp`即可
    >
    > 其他发行版请自行解决

    ```bash
    bash ./build-rootfs-img.sh friendlycore-xenial_4.14_armhf/rootfs friendlycore-xenial_4.14_armhf
    ```

    该命令使用脚本build-rootfs-img.sh以'friendlycore-xenial_4.14_armhf/rootfs'为”原料“在'friendlycore-xenial_4.14_armhf'下生成rootfs.img

    生成以下文件：

    - rootfs.img
    - partmap.txt

    > 使用修改后的rootfs生成的rootfs.img和partmap.txt将会覆盖原来的文件，以便使修改在烧写之后生效

##### NanoPi-R2S

1. 进到NanoPi-R2S的构建工具包sd-fuse_rk3328，将NanoPi-R2S修改后的rootfs的压缩包rootfs-friendlycore-arm64*.tgz解压：

    ```bash
    bsdtar -xzvpf ../images-for-eflasher/friendlycore-arm64-images.tgz
    bsdtar -xzvpf ../rootfs/rootfs-friendlycore-arm64*.tgz
    ```

    > 将*替换为具体文件名

2. 使用`build-rootfs-img.sh`脚本构建rootfs.img：

    > 脚本中用到`tempfile`命令创建临时目录，该命令只有Debian发行版才有（默认自带）
    >
    > Arch Linux发行版对应的命令为`mktemp`，编辑build-rootfs-img.sh文件将第40行`TMPFILE`变量的值改为`mktemp`即可
    >
    > 其他发行版请自行解决

    ```bash
    bash ./build-rootfs-img.sh friendlycore-arm64/rootfs friendlycore-arm64
    ```

    该命令使用脚本build-rootfs-img.sh以'friendlycore-arm64/rootfs'为”原料“在'friendltcore-arm64'下生成rootfs.img

    生成以下文件：

    - rootfs.img
    - partmap.txt
    - param4sd.txt
    - parameter.txt

    > 使用修改后的rootfs生成的rootfs.img、partmap.txt、param4sd.txt和parameter.txt将会覆盖原来的文件，以便使修改在烧写之后生效

#### 3.6 构建可启动TF卡

***该部分全部操作都需要在x86_64架构的Linux系统中进行，即[1.5 需要的系统](#15-需要的系统)中所说的我的电脑***

> 该部分中：
>
> 1. 用到[构建工具包](#12-构建工具包)和[系统文件](#13-系统文件)
> 2. 命令涉及到相对路径的，请以`sd-fuse_*`目录为基准参考[2. 目录结构](#2-目录结构)
> 3. 用到的脚本的参数可以使用`bash <脚本名>`查看

使用`fusing.sh`脚本构建可启动TF卡：

```bash
bash ./fusing.sh /dev/sdX friendlycore-*
```

> `/dev/sdX`：TF卡的设备名
>
> `friendlycore-*`：OS名，如果当前目录中没有与OS同名的文件夹，如果不存在fusing.sh将下载该文件夹

#### 3.7 构建TF卡系统镜像

***该部分全部操作都需要在x86_64架构的Linux系统中进行，即[1.5 需要的系统](#15-需要的系统)中所说的我的电脑***

> 该部分中：
>
> 1. 用到[构建工具包](#12-构建工具包)和[系统文件](#13-系统文件)
> 2. 命令涉及到相对路径的，请以`sd-fuse_*`目录为基准参考[2. 目录结构](#2-目录结构)
> 3. 用到的脚本的参数可以使用`bash <脚本名>`查看

1. 修改脚本

    mk-sd-image.sh脚本默认构建的镜像大小是7800MB，可以修改脚本缩小镜像体积

    编辑mk-sd-image.sh，定位到变量`RAW_SIZE_MB`（位于`friendlycore-xenial_4.14_armhf`分支的），它的默认值是7800（单位MB），将其修改为2500（还可以更小，但太小构建时会出错，具体的值可以自行尝试，只要构建完成没有报错基本就没问题）

2. 构建系统镜像：

    mk-sd-image.sh修改完成之后使用以下命令构建TF卡系统镜像：

    ```bash
    bash ./mk-sd-image.sh friendlycore-* nanopi-r1_sd_friendlycore-xenial_4.14_armhf_Mabo-$(date +%Y%m%d)-NUM.img
    ```

    > `friendlycore-*`：镜像源文件夹名，如果当前目录中没有该文件夹，mk-sd-image.sh将提示下载
    >
    > `nanopi-r1_sd_friendlycore-xenial_4.14_armhf_Mabo-$(date +%Y%m%d)-NUM.img`：自定义的生成的镜像文件名。其中的`-NUM`代表小版本号，例如如果当天构建了两次镜像，则'NUM = 2'

#### 3.8 构建eMMC系统镜像

***该部分全部操作都需要在x86_64架构的Linux系统中进行，即[1.5 需要的系统](#15-需要的系统)中所说的我的电脑***

> 该部分中：
>
> 1. 用到[构建工具包](#12-构建工具包)和[系统文件](#13-系统文件)
> 2. 命令涉及到相对路径的，请以`sd-fuse_*`目录为基准参考[2. 目录结构](#2-目录结构)
> 3. 用到的脚本的参数可以使用`bash <脚本名>`查看

0. 简介信息

    eflasher其实就是一个普通的Ubuntu系统，只是安装了用于烧写系统镜像到eMMC的eflasher包

    eflasher系统信息：

        - 用户名：root
        - 密码：fa
        - 网络（WAN/eth0）：
            - address：192.168.1.231
            - netmask：255.255.255.0
            - gateway：192.168.1.1

1. 修改脚本

    mk-emmc-image.sh会调用mk-sd-image.sh脚本（传递'eflasher'参数）来构建镜像，默认构建的eflasher镜像大小是7800MB，可以修改脚本缩小镜像体积

    编辑mk-sd-image.sh，定位到变量`RAW_SIZE_MB`（位于`eflasher`分支的），它的默认值是7800（单位MB），将其修改为3600（具体的值可以自行尝试，只要构建完成没有报错基本就没问题）

2. 解压文件

    ```bash
    bsdtar -xzvpf ../images-for-eflasher/eflasher.tgz                      # 将eMMC固件文件解压，得到eflasher文件夹
    ```

3. 构建系统镜像：

    修改和解压完成之后使用以下命令构建eMMC系统镜像：

    ```bash
    bash ./mk-emmc-image.sh friendlycore-* nanopi-r1_emmc-eflasher_friendlycore_Mabo-$(date +%Y%m%d)-NUM.img
    ```

    > `friendlycore-*`：镜像源文件夹名，如果当前目录中没有该文件夹，mk-sd-image.sh将提示下载
    >
    > `nanopi-r1_emmc-eflasher_friendlycore_Mabo-$(date +%Y%m%d)-NUM.img`：自定义的生成的镜像文件名。其中的`-NUM`代表小版本号，例如如果当天构建了两次镜像，则'NUM = 2'

4. 开机自动烧写

    eflasher系统镜像构建完成之后即可通过[4 系统烧写](#4-系统烧写)将其写入到TF卡中，将TF卡插入卡槽后上电即可进入eflasher系统，随后可以手动执行`eflasher`命令进行eMMC烧写（具体参数请查看help）

    也可以设置开机自动烧写指定的系统：

    4.1 配置方法

        共有3种配置方法：

        - 方法1：

            将eflasher系统烧写到TF卡之后可以看到TF卡备份为了3个分区，将其挂载之后（不要手动指定挂载点）找到带*FriendlyARM*字样的挂载点并编辑其中的eflasher.conf文件（没有就手动创建）：

            ```toml
            [General]
            autoStart=/mnt/sdcard/XXXXXX
            ```

            其中，XXXXXX替换成TF卡中存放固件的目录名，例如H3平台（例如NanoPi-R1）的FriendlyCore系统目录名为 friendlycore-xenial_4.14_armhf

            一般来说XXXXXX可用的固件名是与eflasher.conf文件同路径下的某一文件夹名，如下图蓝框中所示：

            ![eflasher.conf](https://gitee.com/YJ1516/MyPic/raw/master/picgo/eflasher.png)

        - 方法2：

            在eflasher已经插到NanoPi并上电运行之后，编辑`/mnt/sdcard/eflasher.conf`文件，这样下次上电运行eflasher系统的时候就会自动烧写eMMC

            > 本质和方法1是一样的

        - 方法3：

            在图形界面上，选择要烧写的系统，在'Ready to Go'预览界面上的屏幕下方钩选'Start automatically at startup'

    4.2 烧写完成后自动退出

        eflasher命令烧写完成后不会自动退出，需要修改`eflasher.conf`配置文件（就是上节4.1使用的文件）：

        ```bash
        [General]
        autoExit=true
        ```

    4.3 自动烧写的指示

        NanoPi刚上电时状态灯（对于NanoPi-R1来说是'SYS'灯）呈呼吸灯状态，开始烧写eMMC时呈快闪状态，烧写完成后变为正常状态下的慢闪，此时可以断点后拔下TF卡重新上电，会启动到eMMC里的系统

#### 3.9 后记说明

- 按照[3.6 构建可启动TF卡](#36-构建可启动sd卡)部分即可制作可启动TF卡，但因为分发和批量烧写不方便，需要经[3.7 构建TF卡系统镜像](#37-构建TF卡系统镜像)部分制作成ROM镜像

- 系统镜像构建完成之会在`/Path/to/sd-fuse_*/out`中生成单一img文件（根据系统和构建时间，文件名不同），将其拷贝出来保存之后即可删除`sd-fuse_*`文件夹

- 系统镜像经过压缩之后会大幅缩小空间占用

- NanoPi-R1有8G大小的板载eMMC，如果不使用其作为系统盘的话可以格式化为数据盘：

    1. 执行以下命令查看eMMC设备名：

        ```bash
        lsblk
        ```

        ![eMMC](https://gitee.com/YJ1516/MyPic/raw/master/picgo/emmc.png)

        根据`SIZE`的大小，框线里就是eMMC设备，可以得到eMMC设备名是`/dev/mmcblk1`

    2. 执行以下命令对eMMC进行分区：

        ```bash
        cfdisk /dev/mmcblk1
        ```

        eMMC出厂自带一个OpenWrt系统，所以第一次开机就可以看到它有多个分区（上图是重新分区之后的），使用`cfdisk`命令按照提示对eMMC重新分区

        1. 将eMMC原有分区全部`Delete`，将`Free space`全部`New`成一个区，`Write`之后按`q`键退出cfdisk

        2. 再次使用`lsblk`命令进行查看，如上图eMMC只有一个分区

    3. 执行以下命令格式化eMMC：

        ```bash
        mkfs.ext4 /dev/mmcblk1p1
        ```

        > 格式化为ext4 filesystem

    4. 将eMMC挂载为数据盘：

        1. 创建挂载点`/data`：

            ```bash
            mkdir /data
            ```

        2. 挂载eMMC：

            ```bash
            mount /dev/mmcblk1p1 /data
            ```

        3. 设置开机自动挂载：

            第2步挂载没有问题之后，修改`/etc/fstab`以确保开机自动挂载eMMC

            在`/etc/fstab`末尾**追加**以下行：

            ```bash
            /dev/mmcblk1p1    /data ext4 defaults 0 0
            ```

        保存并重启NanoPi之后，使用`lsblk`命令就会看到`/dev/mmcblk1p1`自动挂载到了`/data`

### 4 系统烧写

#### 4.1 Linux系统下的烧写步骤

1. 解压缩得到ROM镜像（.img文件）

2. 插入TF卡（限4G及以上的卡)，执行`lsblk`命令查看TF卡的设备名（例如`/dev/sdc`）

3. 以root权限执行`dd bs=1M if=/Path/to/img of=/dev/sdX`

    > `if`参数值是镜像文件路径
    >
    > `of`参数值是TF卡设备名
    >
    > `bs`参数不必修改

#### 4.2 Windows系统下的烧写步骤

1. 获取[烧写工具](#14-烧写工具)win32diskimager.exe，解压缩得到ROM镜像（.img文件）

2. 插入TF卡（限4G及以上的卡)，以管理员身份运行烧写工具

3. 在win32diskimager的界面上选择TF卡盘符，选择系统镜像，点击[Write]按钮开始烧写

    ![win32diskimager](https://gitee.com/YJ1516/MyPic/raw/master/picgo/Win32diskimager.jpg)

4. 烧写成功后显示如下界面：

    ![Finish](https://gitee.com/YJ1516/MyPic/raw/master/picgo/Win32disk-finish.jpg)
