---
title: "Debian系的网络配置"
date: 2020-01-15T13:13:34+08:00
showDate: true
showTags: true
showMeta: true
showSocial: true
showActions: true
showPagination: true
categories:
- linux
- network
tags:
- linux
- debian
- network
keywords:
- network
autoThumbnailImage: yes
thumbnailImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/yukari.png
thumbnailImagePosition: left
metaAlignment: center
coverImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/linux_debian_logo_1920x1080.jpg
coverCaption: "KISS"
coverMeta: in
coverSize: full
---

debian系使用NetworkManager配合networking为wifi设置静态IP的办法

<!--more-->

<!-- toc -->

# 起因

最近在捣鼓NanoPi，其官方提供的系统安装了NetworkManager和networking两个网络管理器

默认两个服务都是'enable'状态的，因为功能重叠，我'disable'了networking服务，使用NetworkManager作为默认网络管理器

因为我使用的是NanoPi R1，这个型号有wifi设备，又恰好想要为NanoPi连接的wifi设置静态IP，路由器不方便配置，就想在NanoPi里设置了

# 系统目前的情况

目前，系统中关于网络方面的配置如下：

- {{< alert info no-icon >}}网络管理服务{{< /alert >}}

    - Networkmanager: {{< hl-text green >}}由包network-manager提供{{< /hl-text >}}

- {{< alert info no-icon >}}网络管理工具{{< /alert >}}

    - `nmcli`: {{< hl-text green >}}由包network-manager提供{{< /hl-text >}}
    - `iwlist`: {{< hl-text blue >}}由包wireless-tools提供{{< /hl-text >}}
    - `ifup`: {{< hl-text orange >}}由包ifupdown提供{{< /hl-text >}}
    - `ifdown`: {{< hl-text orange >}}由包ifupdown提供{{< /hl-text >}}

- {{< alert info no-icon >}}网络配置文件{{< /alert >}}

    - `/etc/NetworkManager/NetworkManager.conf`: {{< hl-text green >}}由包network-manager提供{{< /hl-text >}}
    - `/etc/network/interfaces`: {{< hl-text orange >}}由包ifupdown提供{{< /hl-text >}}

由NetworkManager提供网络管理服务，`nmcli`或`iwlist`作为网络搜索工具，`/etc/NetworkManager/NetworkManager.conf`和`/etc/network/interfaces`提供网络配置文件

情况分两种：

1. 需要为wifi配置静态IP

    NetworkManager只提供网络管理和`nmcli`/`iwlist`的基础，wifi设备并不由它管理，具体的网络配置在`/etc/network/interfaces`中进行

2. 不需要为wifi配置静态IP，由路由器DHCP自动分配

    NetworkManager提供网络管理和`nmcli`/`iwlist`的基础并管理wifi设备，使用`nmcli`命令连接wifi

# 准备工作

{{< alert info >}}这种情况下不需要ifupdown{{< /alert >}}

## 查看NanoPi的网络设备

执行以下命令：

{{< codeblock "shell" "shell" >}}
$ nmcli dev
{{< /codeblock >}}

{{< image classes="fancybox left clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/nmcli_dev.png" thumbnail="https://gitee.com/YJ1516/MyPic/raw/master/picgo/nmcli_dev.png" group="group:travel" thumbnail-width="692px" thumbnail-height="147px" title="查看网络设备" >}}

> 各列的含义：
>
> DEVICE列：设备名
>
> TYPE列：设备种类
>
> STATE列：连接/管理状态
>
> CONNECTION列：简要连接信息

## 开启wifi设备

如果'wlan0'的STATE值是'uavailable'，代表wifi设备不可用，使用以下命令激活wifi设备：

{{< codeblock "shell" "shell" >}}
$ nmcli r wifi on
{{< /codeblock >}}

## 扫描周围wifi信息

执行以下命令：

{{< codeblock "shell" "shell" >}}
$ nmcli dev wifi
{{< /codeblock >}}


{{< image classes="fancybox left clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/wifi_info.png" thumbnail="https://gitee.com/YJ1516/MyPic/raw/master/picgo/wifi_info.png" group="group:travel" thumbnail-width="1375px" thumbnail-height="146px" title="扫描wifi" >}}

# 连接wifi

## 不需要为wifi配置静态IP，由路由器DHCP自动分配

因为不需要修改wifi连接参数，所以使用`nmcli`命令即可

[扫描周围wifi信息](#扫描周围wifi信息)完成之后，如图所示：

{{< image classes="fancybox left clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/wifi_info.png" thumbnail="https://gitee.com/YJ1516/MyPic/raw/master/picgo/wifi_info.png" group="group:travel" thumbnail-width="1375px" thumbnail-height="146px" title="扫描wifi" >}}

根据wifi的'SSID'（即wifi名）进行连接，执行以下命令：

```shell
# nmcli dev wifi connect "SSID" password "PASSWORD"
```

> "SSID"是wifi名，"PASSWORD"是对应的密码
>
> 连接成功之后，每次重启NanoPi都会自动连接该wifi

如果Pi有多个无线网络设备，可以使用ifname参数指定使用的设备名：

{{< codeblock "shell" "shell" >}}
# nmcli dev wifi connect "SSID" password "PASSWORD" ifname "wlanX"
{{< /codeblock >}}

## 需要为wifi配置静态IP

因为需要修改wifi的连接参数，所以需要配合ifupdown的网络配置文件`/etc/network/interfaces`一起使用

### 修改配制文件

#### NetworkManager的配置文件

确保NetworkManager的配置文件`/etc/NetworkManager/NetworkManager.conf`中有如下配置：

{{< codeblock "shell" "shell" >}}
[ifupdown]
managed=false
{{< /codeblock >}}

**`managed`的值必须是`false`**

#### ifupdown的网络配制文件

[扫描周围wifi信息](扫描周围wifi信息)完成之后，如图所示：

{{< image classes="fancybox left clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/wifi_info.png" thumbnail="https://gitee.com/YJ1516/MyPic/raw/master/picgo/wifi_info.png" group="group:travel" thumbnail-width="1375px" thumbnail-height="146px" title="扫描wifi" >}}

'SSID'列是wifi名，'CHAN'列是wifi使用的信道，需要用到这两个参数

修改ifupdown的网络配置文件`/etc/network/interfaces`，在文件的最后空一行新增如下内容:

{{< codeblock "shell" "shell" >}}
allow-hotplug <无线设备名>
iface <无线设备名> inet static
address <静态IP>
netmask <子网掩码>
wpa-ssid <SSID>
wpa-passphrase <wifi密码>
wireless-channel <Channel>
{{< /codeblock >}}

> <无线设备名>：如图所示，'DEVICE'列就是网络设备的名字，从中确定无线设备名，手动设置
>
> {{< image classes="fancybox left clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/nmcli_dev.png" thumbnail="https://gitee.com/YJ1516/MyPic/raw/master/picgo/nmcli_dev.png" group="group:travel" thumbnail-width="692px" thumbnail-height="147px" title="查看网络设备" >}}
>
> <静态IP>：要连接的wifi的IP，手动设置
>
> <子网掩吗>：要连接的wifi的子网掩码，手动设置
>
> <SSID>：要连接的wifi的名字，手动设置
>
> <wifi密码>：要连接的wifi的密码，手动设置
>
> <Channel>：要连接的wifi的信道，手动设置。如图，'CHAN'列即是各个wifi对应的信道

### 使配置生效

配置文件修改完成后，执行以下命令使配置文件生效：

1. 重启NetworkManager：

    {{< codeblock "shell" "shell" >}}
    # systemctl restart NetworkManager
    {{< /codeblock >}}

2. 启动配置好的无线设备

    {{< codeblock "shell" "shell" >}}
    # ifup <无线设备名>
    {{< /codeblock >}}

    > 例如：`ifup wlan0`

### 查看是否生效

执行以下命令：

{{< codeblock "shell" "shell" >}}
$ ip a
{{< /codeblock >}}

正常情况下可以看到无线设备已经设置为指定的IP，并且可以ping通同一wifi下的其他设备
