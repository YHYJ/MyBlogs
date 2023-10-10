---
title: "解决ssh连接用时过长问题"
date: 2019-10-17T15:01:16+08:00
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
- ssh
- fix
keywords:
- ssh
- fix
autoThumbnailImage: yes
thumbnailImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/Qmei.jpg
thumbnailImagePosition: left
metaAlignment: center
coverImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/light_sky.jpg
coverCaption: "黑暗是生命和文明之母"
coverMeta: in
coverSize: full
---

某些Linux发行版默认ssh配置会导致连接用时过长，修改一个配置项即可修复该问题......

<!--more-->

<!-- toc -->



# 修改配置文件

ssh的服务端程序是`sshd`，它的默认配置文件是`/etc/ssh/sshd_config`

1. 定位到如下行

    {{< codeblock >}}
    ...
    UseDNS yes
    ...
    {{< /codeblock >}}

    {{< alert info >}}`UseDNS`选项用来指定是否应该对远程主机名进行反向解析以检查此主机名是否与其IP地址真实对应，这是一个耗时操作{{< /alert >}}

    该行可能是注释(开头是`#`)的，但只要`UseDNS`后面的值是`yes`，就代表该功能是开启状态

2. 关闭`UserDNS`

    将`UseDNS`的值修改为`no`

    {{< codeblock >}}
    ...
    UseDNS no
    ...
    {{< /codeblock >}}

---

# 重启服务

执行以下命令重启`sshd`服务：

{{< codeblock "bash" "bash">}}
systemctl restart sshd
{{< /codeblock >}}

---

OK，重启以后再次ssh连接该机器就不会出现耗时过长的问题了
