---
title: "先装Linux后装Windows的引导问题"
date: 2020-11-01T16:39:00+08:00
showDate: true
showTags: true
showMeta: true
showSocial: true
showActions: true
showPagination: true
categories:
- System
- Bootloader
tags:
- system
- bootloader
- linux
- windows
keywords:
- Bootloader
autoThumbnailImage: yes
thumbnailImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/20201101_165119.png
thumbnailImagePosition: top
metaAlignment: center
coverImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/Arch_1920x1080.jpeg
coverCaption: "Arch Linux"
coverMeta: in
coverSize: full
---

双系统(Linux/Windows)环境下重装Windows的注意事项

<!--more-->

系统：

- Arch Linux
    - Bootloader：systemd-boot
- Windows 10
    - Bootloader：Windows Boot Manager

名词解释：

- ESP：即EFI System Partition，EFI系统分区
- bootloader：bootstrap loader，引导加载程序

<!-- toc -->

# 关于bootloader

Linux下有多款bootloader可选，包括systemd-boot、grub等，而Windows只有自带的bootloader叫做Windows Boot Manager

# systemd-boot

systemd-boot属于systemd的一个组件，因为系统/服务管理器使用的是systemd，不用像使用grub一样还要单独安装软件包

`bootctl`命令用于安装/更新/删除systemd-boot条目(entry)，常用参数有：

```bash
## Boot Loader本身相关命令
# 在ESP和EFI变量中安装systemd-boot
bootctl install
# 更新ESP和EFI变量中的systemd-boot
bootctl update
# 从ESP和EFI变量中删除systemd-boot
bootctl remove
# 显示已安装的systemd-boot和EFI变量的状态
bootctl status

## Boot Loader条目相关命令
# 列出所有条目
bootctl list
# 设置默认引导的条目
bootctl set-default ID
```

# 先装Linux后装Windows时对ESP的要求

安装Linux的时候需要设置ESP，该分区需要满足以下条件：

- 分区类型为**EFI System**（中文环境下是**EFI 系统**）

    > ID = 1
    >
    > UUID = C12A7328-F81F-11D2-BA4B-00A0C93EC93B

- 文件系统是**vfat**

- 挂载点最好是`/boot`

- 大小尽量在150M～500M之间

以上条件前两条满足时再安装Windows即可自动将其Bootloader放到该分区（注意安装Windows的时候不要把这个分区格式化了）

> 第3条是Linux的要求（但不是强制的）
>
> 第4条是个人建议，如果以后在Linux下要用`fwupd`更新固件，因为`fwupd`是把新固件包缓存到ESP下的，分区太小的话很容易更新失败

# 对Windows Boot Manager的处理

Windows“流氓”在安装完成后会自动调整BIOS的启动顺序，把Windows Boot Manager设置成第1位，导致开机自动进入Windows系统而不会进到系统选择界面

只要在开机的时候进到BIOS，把ESP所在的硬盘调整为第1位即可
