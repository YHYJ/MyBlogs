---
title: "解决sddm主题报QtQuick.Controls未安装的错误"
date: 2019-10-29T12:18:15+08:00
showDate: true
showTags: true
showMeta: true
showSocial: true
showActions: true
showPagination: true
categories:
- bug
- fix
tags:
- fix
- sddm
- qt
keywords:
- sddm
- linux
autoThumbnailImage: yes
thumbnailImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/nebula1.jpg
thumbnailImagePosition: top
metaAlignment: center
coverImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/1.jpg
coverCaption: "我独处时最轻松，因为我不觉得自己乏味，即使乏味，也自己承受，不累及他人，无需感到不安"
coverMeta: in
coverSize: full
---

当sddm报`QtQuick.Controls`错误的时候，后期设置的sddm theme就会失效，变为默认theme（chiou）

<!--more-->

<!-- toc -->

# 前言

{{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/info.png" thumbnail="" group="group:system-info" thumbnail-width="100%" thumbnail-height="100%" title="system info" >}}

将系统切换到Arch Linux并使用[i3-wm](https://i3wm.org/)，因为刚开始不熟悉i3，装了KDE全家桶和plasma-desktop来配置fcitx5、theme和font，sddm的theme就是在plasma下安装配置的

---

# 起因

熟悉i3后，就把KDE和plasma-desktop卸载了（空出了2G+的空间）

但是！系统重启进入到sddm主界面的时候，theme变成了默认的，并且报了个Bug：

{{< codeblock "BUG" "c" >}}
... module "QtQuick.Controls" version <version> is not installed
{{< /codeblock >}}

---

# 经过

看报错像是有一个包没有安装，关键字是**qtquick**

1. `pacman`搜索qtquick：

    {{< codeblock "bash" "bash" >}}$ pacman -Ss qtquick{{< /codeblock >}}

    找到了这个：
    {{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/kirigami2.png" thumbnail="" group="group:pacman" thumbnail-width="100%" thumbnail-height="100%" title="kirigami2" >}}

    **kirigami2**是KDE的一个QtQuick基础组件包

2. 查看kirigami2的详细信息：

    {{< codeblock "bash" "bash" >}}$ pacman -Si kirigami2{{< /codeblock >}}

    {{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/kirigami2info.png" thumbnail="" group="group:pacman" thumbnail-width="100%" thumbnail-height="100%" title="kirigami2 info" >}}

    可以看到，kirigami2依赖了3个包，其中**qt5-quickcontrols2**和报错关键字符合

3. 查看qt5-quickcontrols2的详细信息：

    {{< codeblock "bash" "bash" >}}$ pacman -Si qt5-quickcontrols2{{< /codeblock >}}

    {{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/qt5-quickcontrols2_info.png" thumbnail="" group="group:pacman" thumbnail-width="100%" thumbnail-height="100%" title="qt5-quickcontrols2 info" >}}

    是一个基于Qt Quick的界面控件，完全符合报错信息

4. 安装qt5-quickcontrols2：

    {{< codeblock "bash" "bash" >}}# pacman -S qt5-quickcontrols2{{< /codeblock >}}

---

# 结果

安装好qt5-quickcontrols2以后，重启机器，sddm终于恢复正常了：

{{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/sddm_theme.png" thumbnail="" group="group:sddm" thumbnail-width="100%" thumbnail-height="100%" title="Red-Black" >}}

---

# 后记

{{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/sddm.png" thumbnail="" group="group:sddm" thumbnail-width="100%" thumbnail-height="100%" title="sddm package" >}}

从上图来看，sddm不依赖于qt5-quickcontrols2，不知道具体为什么会因为没有安装qt5-quickcontrols2导致自定义sddm theme失效，因为当时是在KDE和plasma-desktop存在的情况下设置的sddm theme，可能会有影响？

由于机器自带的Samsung SSD走的是PCIe接口（并且固态是焊接到主板上的），预留的是SATA接口，虽然理论速度两者相差无几，但是一是自带的是PCIe x4，二是PCIe离CPU更近，导致虽然自己加的Intel SSD性能和自带的Samsung持平甚至可能略有超出，但实际测试还是没有自带的快，又由于系统是装在Intel上的，并不能发挥出系统的全速，所以考虑重装系统到Samsung，重装后的系统就不会再安装KDE全家桶了，到时候看看情况吧
