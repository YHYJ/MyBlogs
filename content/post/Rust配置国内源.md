---
title: "Rust配置国内源"
date: 2019-11-07T11:45:02+08:00
showDate: true
showTags: true
showMeta: true
showSocial: true
showActions: true
showPagination: true
categories:
- rust
- configure
tags:
- language
- rust
- 换源
keywords:
- language
autoThumbnailImage: yes
thumbnailImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/cat_glance_gray_150707_750x236.jpg
thumbnailImagePosition: top
metaAlignment: center
coverImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/cat_paws_furry_113514_1920x1080.jpg
coverCaption: "蒹葭苍苍 白露为霜"
coverMeta: in
coverSize: full
---

rust的默认源位于国外，使用起来慢的很，要提升使用体验，就要更换为国内源......

<!--more-->

<!-- toc -->

# 说明

这里涉及到两个rust组件的换源：一个是工具链管理器[rustup](http://www.rustup.rs/)，一个是包管理器[cargo](https://crates.io/)

{{< alert info >}}这里只记录在Linux使用bash/zsh更换rust源的方法，不涉及其他平台/shell{{< /alert >}}

---

# rustup

rustup需要设置{{< hl-text primary >}}两个源{{< /hl-text >}}，一个用于更新toolchain，一个用于更新rustup

- toolchain

    - 针对当前用户

        在`.bashrc`/`.zshrc`文件中插入`RUSTUP_DIST_SERVER`变量：

        {{< codeblock "bash" "bash" >}}export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static{{< /codeblock >}}

    - 针对所有用户

        新建文件`/etc/profile.d/rust.sh`，在其中插入`RUSTUP_DIST_SERVER`变量：

        {{< codeblock "bash" "bash" >}}export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static{{< /codeblock >}}

- rustup

    - 针对当前用户

        在`.bashrc`/`.zshrc`文件中插入`RUSTUP_UPDATE_ROOT`变量：

        {{< codeblock "bash" "bash" >}}export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup{{< /codeblock >}}

    - 针对所有用户

        新建文件`/etc/profile.d/rust.sh`，在其中插入`RUSTUP_UPDATE_ROOT`变量：

        {{< codeblock "bash" "bash" >}}export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup{{< /codeblock >}}

---

# cargo

{{< alert info >}}这里默认cargo使用默认安装路径`$HOME/.cargo`{{< /alert >}}

新建`$HOME/.cargo/config`文件，在其中插入以下内容:

{{< codeblock "toml" "toml" >}}
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
{{< /codeblock >}}

{{< alert info >}}如果当前不支持git协议，将`registry`的值替换为`"http://mirrors.ustc.edu.cn/crates.io-index"`{{< /alert >}}

如果cargo的版本小于'0.13.0'，在`$HOME/.cargo/config`中插入以下内容:

{{< codeblock "toml" "toml" >}}
[registry]
index = "git://mirrors.ustc.edu.cn/crates.io-index"
{{< /codeblock >}}

{{< alert info >}}如果当前不支持git协议，将`index`的值替换为`"http://mirrors.ustc.edu.cn/crates.io-index"`{{< /alert >}}

---

# 后记

环境变量和配置文件更新完成后，打开一个新的终端使环境变量生效，此时安装/更新toolchain/crate的速度就会大大提升
