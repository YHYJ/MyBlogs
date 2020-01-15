---
title: "Grafana调整Panel的Title字体大小"
date: 2019-11-01T13:30:42+08:00
showDate: true
showTags: true
showMeta: true
showSocial: true
showActions: true
showPagination: true
categories:
- visualization
- grafana
tags:
- grafana
- font
keywords:
- grafana
- panel
- title
- font
- size
autoThumbnailImage: yes
thumbnailImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/urheeneggs.jpg
thumbnailImagePosition: left
metaAlignment: center
coverImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/grafana.jpg
coverCaption: "If you miss the train I'm on, You will know that I am gone"
coverMeta: in
coverSize: full
---

grafana中panel的title默认字体很小，特别是投放到大屏上之后，基本处于看不清的状态

<!--more-->

<!-- toc -->

在PC上grafana展示效果挺不错的，但一旦投到大屏上，panel title的字体就显得过小了，万一大屏分辨率再不高(不到1080p)，基本是一团墨汁，因此需要修改title的字体大小

经过一番搜索，在[grafana官方仓库](https://github.com/grafana/grafana)的[{{< hl-text green >}}#1875{{< /hl-text >}}](https://github.com/grafana/grafana/issues/1875)找到了解决方案，结合官方[CHANGELOG](https://github.com/grafana/grafana/blob/v6.0.0/CHANGELOG.md#breaking-changes-1)基本算是完美解决

# 注意

该解决方案是通过在Text panel中注入html来实现的，根据[#4117](https://github.com/grafana/grafana/issues/4117)所说，允许在Text panel里注入html是有安全隐患的，因此最晚在v6.0.0版本更新中，官方默认禁用了该功能

文章是通过在Text panel注入html来实现修改title字体的，因此没有能力承担该操作带来的风险的话，请不要进行操作！

---

# 修改grafana配置项

根据v6.0.0的CHANGELOG中[breaking-changes-1](https://github.com/grafana/grafana/blob/v6.0.0/CHANGELOG.md#breaking-changes-1)条目所说，自该版本之后grafana默认禁用了html注入，有两种方法解除禁用：

- 修改grafana配置文件
    {{< alert info >}}在docker中，grafana的配置文件是`/usr/share/grafana/conf/defaults.ini`{{< /alert >}}

    打开`defaults.ini`，找到如图所示字段：

    {{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/grafana_confile.png" thumbnail="" group="group:grafana-confile" thumbnail-width="100%" thumbnail-height="100%" title="grafana confile" >}}

    将其中的`disable_sanitize_html`的值修改为`true`

- 传递环境变量（推荐）

    container启动时定义的环境变量会覆盖配置文件，所以推荐使用环境变量的形式进行设置

    {{< codeblock "yaml" "yaml" >}}
    ...
    services:
        grafana:
        image: 'grafana/grafana'
        container_name: 'grafana'
        restart: 'unless-stopped'    # no, unless-stopped, always, on-failure:<max-retries>
        privileged: false
        ports:
            - '3000:3000'
        volumes:
            - 'grafana:/var/lib/grafana'
        networks:
            data: {}
        environment:
            TZ: 'Asia/Shanghai'
            GF_PANELS_DISABLE_SANITIZE_HTML: 'true'     # 启用html注入
    ...
    {{< /codeblock >}}

    {{< alert info >}}使用`docker run`而非Compose的话用`-e`参数设置环境变量{{< /alert >}}

配置文件或者compose file修改完成以后存在正在运行的grafana container的话则需要重新启动

---

# 修改title字体

确保修改后的配置项/环境变量作用到grafana后，接下来是修改Panel Title的字体

1. 在dashboard中新增一个Text类型的Panel，将'Visualization'-'Mode'的值改为'html'

{{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/add_text_panel.png" thumbnail="" group="group:grafana" thumbnail-width="100%" thumbnail-height="100%" title="Add Text Panel" >}}

2. 将如下CSS代码写入文本框

{{< codeblock "css" "css" >}}
<style>
.panel-title-text {
font-size: 30px;
}
.panel-container {
height: 100%;
}
</style>
{{< /codeblock >}}

{{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/add_css_code.png" thumbnail="" group="group:grafana" thumbnail-width="100%" thumbnail-height="100%" title="Add CSS Code" >}}

3. 最小&透明化Text Panel

因为这个Text Panel只是用来设置panel title的，所以要尽量让它“隐身”

将'General'按照如下图所示配置：

{{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/modify_general.png" thumbnail="" group="group:grafana" thumbnail-width="100%" thumbnail-height="100%" title="Modify General" >}}

配置完以后返回到dashboard界面，将Text Panel的大小调到最小，然后找个角落“藏着”它即可

{{< image classes="fancybox center clear" src="https://gitee.com/YJ1516/MyPic/raw/master/picgo/ok.png" thumbnail="" group="group:grafana" thumbnail-width="100%" thumbnail-height="100%" title="OK" >}}

不论是添加Text Panel之前还是之后的Panel，它们的Title字体都会变大，如上图所示（红色框里的就是“隐身”的Text Panel）
