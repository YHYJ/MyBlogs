---
title: "调整docker的log配置参数"
date: 2019-11-04T10:13:11+08:00
showDate: true
showTags: true
showMeta: true
showSocial: true
showActions: true
showPagination: true
categories:
- 容器
- docker
tags:
- docker
- 容器
- compose
keywords:
- tech
autoThumbnailImage: yes
thumbnailImage: https://i.loli.net/2019/11/05/6gjLtWly2uoq1UH.jpg
thumbnailImagePosition: left
metaAlignment: center
coverImage: https://i.loli.net/2019/11/04/wSrfcARzQp9ymqn.jpg
coverCaption: "活下去"
coverMeta: in
coverSize: full
---

docker默认的配置并不会限制log文件的大小，这导致了我的NanoPi-R1自带的eMMC(8G)在短短4天内就被log文件撑满了......

<!--more-->

<!-- toc -->

# 前言

我把系统烧写到了一张16G的TF卡上，{{< hl-text success >}}NanoPi-R1{{< /hl-text >}}（以下简称R1）自带的eMMC(8G)挂载到``/data``专门用来存放docker数据（image、container、volume、log...），
因为R1是有特定用途的，所以里面只有{{< hl-text cyan >}}redis:latest{{< /hl-text >}}、{{< hl-text cyan >}}influxdb:latest{{< /hl-text >}}和一个{{< hl-text cyan >}}自制的350M+的image{{< /hl-text >}}，总大小在{{< hl-text red >}}600-700M{{< /hl-text >}}之间，{{< hl-text red >}}剩余6G{{< /hl-text >}}的空间用来存放docker的其他数据，我原以为是完全够用的……

---

# 经过

周五下午，我在R1上跑了5个Compose，分别从5个IP获取数据，每个Compose有3个container，这5个Compose除了数据源的IP外完全一样，每个Compose都有grafana监控页面

周末完全没有管它……

周一下午，看了一眼grafana，数据正常……

周二下午，grafana上只有#5 Compose有数据了！ssh连接R1也是失败的，只好直接断电才登录了上去，`df -h`一看
{{< alert warning >}}eMMC已经被塞满了{{< /alert >}}
因为eMMC上只有docker的数据，所以直接从docker查找原因

## 查找原因

{{< alert info no-icon >}}当时排查R1的时候忘了截图，只能看示例图了……{{< /alert >}}

### docker命令查看

运行：

{{< codeblock "bash" "bash" >}}
docker system df
{{< /codeblock >}}

能够得到Images、Containers、Local Volumes、Build Cache的TOTAL、ACTIVE、SIZE、RECLAIMABLE信息

或者运行：

{{< codeblock "bash" "bash" >}}
docker ps -s
{{< /codeblock >}}

可以单独查看container的SIZE

输出如下：

{{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/docker_system_df_r1.png" thumbnail="" group="group:docker" thumbnail-width="100%" thumbnail-height="100%" title="docker system df" >}}

从上图可知docker占用的空间并不多

这种情况明显是这两个命令没有查出docker到底在哪里占用了大量空间

docker命令查找不出来，只能直接查看docker的文件夹了

### 直接查看docker文件夹

运行以下命令按大小排序docker文件夹：

{{< codeblock "bash" "bash" >}}
du -h --max-depth=1 /data/docker | sort -h
{{< /codeblock >}}

输出如下：

{{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/du_docker.png" thumbnail="" group="group:du" thumbnail-width="100%" thumbnail-height="100%" title="du" >}}

其中**overlay2**是docker使用的文件系统，不要碰它

在我的R1中，containers文件夹大小达到了4G+，这显然是不正常的，考虑到3个container中有1个会输出大量log，怀疑是log文件占用了大量空间

到`/data/docker/containers/XXXXXX`文件夹下一看，果然有一个名为`XXXXXX-json.log`的文件达到了1G左右

{{< alert info >}}其中XXXXXX是对应container的ID{{< /alert >}}

这样1×5=5G，加上overlay2和images占用的空间，eMMC已经完全满了！

---

# 解决

确定了问题的原因，只要限制一下docker的log的大小即可

官方文档[docker logging configure](https://docs.docker.com/config/containers/logging/configure/)对docker log的配置进行了解释

## docker支持的log驱动

| Driver       | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| `none`       | 不启用log功能，`docker logs`不会有任何输出                   |
| `local`      | 使用自定义格式的log,旨在最大限度的减少开支                   |
| `json-file`  | JSON格式log，这是docker的默认log驱动                         |
| `syslog`     | Linux的系统log服务，必须保证syslogd在宿主机上已运行          |
| `journald`   | systemd的log服务，可以代替syslog服务                         |
| `gelf`       | 将log写入graylog端点                                         |
| `fluentd`    | 将log写入fluentd，必须保证fluentd在宿主机上已运行            |
| `awslogs`    | 将log写入Amazon CloudWatch Logs                              |
| `splunk`     | 使用HTTP Event Collector将log写入splunk                      |
| `etwlogs`    | 将log写为Event Tracing for Windows (ETW)事件，仅适用于Windows平台 |
| `gcplogs`    | 将log写入Google Cloud Platform (GCP)                         |
| `logentries` | 将log写入Rapid7 Logentries                                   |

## 设置log驱动

docker默认的log驱动是json-file，使用以下命令查看当前log驱动：

{{< codeblock "bash" "bash" >}}
docker info --format '{{.LoggingDriver}}'
{{< /codeblock >}}

有2种方法设置docker的log驱动（还有一种配置dockerd的方法，但极其不推荐，所以不写）：

1. 配置daemon.json（不推荐这种方法，不灵活）

    文件路径：

    {{< hl-text success >}}Linux：`/etc/docker/daemon.json`{{< /hl-text >}}
    \
    {{< hl-text primary >}}Windows：`C:\ProgramData\docker\config\daemon.json`{{< /hl-text >}}

    如果使用的驱动有可配置项也可以在daemon.json里配置：

    {{< codeblock "json" " json" >}}
    {
        "log-driver": "json-file",
        "log-opts": {
        "max-size": "10m",
        "max-file": "3",
        "labels": "production_status",
        "tag": "{{.Name}}/{{.ID}}/{{.ImageName}}/{{.ImageID}}"
    }
    {{< /codeblock >}}

    以上示例中：

    - `log-driver`：设置驱动为json-file
    - `log-opts`是驱动的配置项
    - `max-size`：设置单个log文件的最大尺寸
    - `max-file`：设置log文件的最大数量
    - `labels`：设置log的label，用于log收集工具的数据处理
    - `tag`：指定按照什么格式在log中记录container本身的的信息，详见[logging/log_tags](https://docs.docker.com/v17.09/engine/admin/logging/log_tags/)

2. 配置docker0compose.yml（推荐，为每个service灵活定制log选项）

    在每个service下添加如下代码，即可定制每个service的log选项：

    {{< codeblock "json" " json" >}}
    ...
        labels:
            <label_name>: "<label_value>"
        logging:
            driver: "json-file"
            options:
            labels: "<label_name>"
            tag: "{{.Name}}/{{.ID}}/{{.ImageName}}/{{.ImageID}}"
            max-size: "10m"
            max-file: "10"
    ...
    {{< /codeblock >}}
