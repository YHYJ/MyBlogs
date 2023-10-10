---
title: "Shell脚本输出控制码"
date: 2019-11-25T15:49:25+08:00
showDate: true
showTags: true
showMeta: true
showSocial: true
showActions: true
showPagination: true
categories:
- shell
- color
tags:
- shell
- output
- color
keywords:
- shell
autoThumbnailImage: yes
thumbnailImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/girl_glance_fence_151765_540x960.jpg
thumbnailImagePosition: left
metaAlignment: center
coverImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/girl.jpg
coverCaption: "I'm not good, not bad, but I sure as hell ain't ugly"
coverMeta: in
coverSize: full
---

控制Shell脚本输出内容的格式，包括颜色、背景色、属性、光标......

<!--more-->

<!-- toc -->

# 字符颜色码

{{< hl-text red >}}
红色 - 31
{{< /hl-text >}}

{{< hl-text green >}}
绿色 - 32
{{< /hl-text >}}

{{< hl-text yellow >}}
黄色 - 33
{{< /hl-text >}}

{{< hl-text blue >}}
蓝色 - 34
{{< /hl-text >}}

{{< hl-text purple >}}
紫色 - 35
{{< /hl-text >}}

{{< hl-text cyan >}}
青色 - 36
{{< /hl-text >}}

- 黑色 - `30`
- 白色 - `37`
- 重置 - `0`

# 背景颜色码

{{< hl-text red >}}
红色 - 41
{{< /hl-text >}}

{{< hl-text green >}}
绿色 - 42
{{< /hl-text >}}

{{< hl-text yellow >}}
黄色 - 43
{{< /hl-text >}}

{{< hl-text blue >}}
蓝色 - 44
{{< /hl-text >}}

{{< hl-text purple >}}
紫色 - 45
{{< /hl-text >}}

{{< hl-text cyan >}}
青色 - 46
{{< /hl-text >}}

- 黑色 - `40`
- 白色 - `47`
- 重置 - `0`

# 属性控制码

- 关闭所有属性 - `0`
- 加粗 - `1`
- 下划线 - `4`
- 闪烁 - `5`
- 反显 - `7`
- 消隐 - `8`

# 光标控制码

- 光标上移n行 - `nA`
- 光标下移n行 - `nB`
- 光标右移n行 - `nC`
- 光标左移n行 - `nD`
- 设置光标位置 - `y;xH`
- 清屏 - `2J`
- 清楚从光标到行尾的内容 - `K`
- 保存光标位置 - `s`
- 恢复光标位置 - `u`
- 隐藏光标 - `?25l`
- 显示光标 - `?25h`
