---
title: "Hello World"
date: 2019-10-16T10:27:39+08:00
showDate: true
showTags: true
showMeta: true
showSocial: true
showActions: true
showPagination: true
categories:
- 杂记
- 创世纪
tags:
- Genesis
keywords:
- genesis
autoThumbnailImage: yes
thumbnailImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/Winston.png
thumbnailImagePosition: top
metaAlignment: center
coverImage: https://gitee.com/YJ1516/MyPic/raw/master/picgo/0-0.jpg
coverCaption: "永远不要只满足于世界的表象，要敢于探寻未知的可能"
coverMeta: in
coverSize: full
---
文章摘要...
<!--more-->

<!-- toc -->


# 原生Markdown语法

- 目录

    # 一级目录

    ## 二级目录

    ### 三级目录

    #### 四级目录

    ##### 五级目录

    ###### 六级目录

- 列表

    - List 1
    - List 2
    - ...
    - List n

- 注释

    > 注释
    >
    > 缩略图最佳长宽是750x236

---

# Theme附带语法

- 代码

    -  高亮代码块

        {{< codeblock "go" "go" >}}
        package main

        import "fmt"

        func main() {
            fmt.Println("Hello World")
        }
        {{< /codeblock >}}

        {{< codeblock "python" "python" >}}
        print("Hello World")
        {{< /codeblock >}}

        {{< codeblock "bash" "bash" >}}
        echo 'Hello World'
        {{< /codeblock >}}

        {{< codeblock "rust" "rust" >}}
        fn main() {
            let greetings = [
                "Hello",
                "Hola",
                "Bonjour",
                "Ciao",
                "こんにちは",
                "안녕하세요",
                "Cześć",
                "Olá",
                "Здравствуйте",
                "Chào bạn",
                "您好",
                "Hallo",
                "Hej",
                "Ahoj",
                "سلام",
                "สวัสดี",
            ];

            for (num, greeting) in greetings.iter().enumerate() {
                print!("{} : ", greeting);
                match num {
                    0 => println!("This code is editable and runnable!"),
                    1 => println!("¡Este código es editable y ejecutable!"),
                    2 => println!("Ce code est modifiable et exécutable !"),
                    3 => println!("Questo codice è modificabile ed eseguibile!"),
                    4 => println!("このコードは編集して実行出来ます！"),
                    5 => println!("여기에서 코드를 수정하고 실행할 수 있습니다!"),
                    6 => println!("Ten kod można edytować oraz uruchomić!"),
                    7 => println!("Este código é editável e executável!"),
                    8 => println!(
                        "Этот код можно отредактировать и запустить!"
                    ),
                    9 => println!("Bạn có thể edit và run code trực tiếp!"),
                    10 => println!("这段代码是可以编辑并且能够运行的！"),
                    11 => println!("Dieser Code kann bearbeitet und ausgeführt werden!"),
                    12 => println!("Den här koden kan redigeras och köras!"),
                    13 => println!("Tento kód můžete upravit a spustit"),
                    14 => println!("این کد قابلیت ویرایش و اجرا دارد!"),
                    15 => println!(
                        "โค้ดนี้สามารถแก้ไขได้และรันได้"
                    ),
                    _ => {}
                }
            }
        }
        {{< /codeblock >}}

    -  选项卡式代码块

        {{< tabbed-codeblock 选项卡式代码块 >}}
        <!-- tab bash -->
            echo 'Hello World'
        <!-- endtab -->
        <!-- tab python -->
            print('Hello World')
        <!-- endtab -->
        {{< /tabbed-codeblock >}}

- 标签

    {{< alert info no-icon >}}这是一个没有图标的信息标签{{< /alert >}}

    {{< alert info >}}这是一个信息标签{{< /alert >}}

    {{< alert success >}}这是一个成功标签{{< /alert >}}

    {{< alert warning >}}这是一个告警标签{{< /alert >}}

    {{< alert danger >}}这是一个危险标签{{< /alert >}}\

- 文本高亮

    {{< hl-text red >}}红色高亮文本{{< /hl-text >}}
    {{< hl-text orange >}}橙色高亮文本{{< /hl-text >}}
    {{< hl-text yellow >}}黄色高亮文本{{< /hl-text >}}
    {{< hl-text green >}}绿色高亮文本{{< /hl-text >}}
    {{< hl-text cyan >}}青色高亮文本{{< /hl-text >}}
    {{< hl-text blue >}}蓝色高亮文本{{< /hl-text >}}
    {{< hl-text purple >}}紫色高亮文本{{< /hl-text >}}

    {{< hl-text primary >}}首要高亮文本{{< /hl-text >}}
    {{< hl-text success >}}成功高亮文本{{< /hl-text >}}
    {{< hl-text warning >}}告警高亮文本{{< /hl-text >}}
    {{< hl-text danger >}}危险高亮文本{{< /hl-text >}}

- 图片

    - 可点击图片

        {{< image classes="fancybox left clear" src="https://i.loli.net/2019/11/04/C7bRGFrMuLcJ4Ww.jpg" thumbnail="https://i.loli.net/2019/11/04/C7bRGFrMuLcJ4Ww.jpg" group="group:travel" thumbnail-width="250px" thumbnail-height="250px" title="可点击图片-左对齐" >}}

        {{< image classes="fancybox right clear" src="https://i.loli.net/2019/11/05/6gjLtWly2uoq1UH.jpg" thumbnail="https://i.loli.net/2019/11/05/6gjLtWly2uoq1UH.jpg" group="group:travel" thumbnail-width="250px" thumbnail-height="250px" title="可点击图片-右对齐" >}}

        {{< image classes="fancybox center clear" src="https://i.loli.net/2019/11/05/WCiUr65LuHG29gT.jpg" thumbnail="https://i.loli.net/2019/11/05/WCiUr65LuHG29gT.jpg" group="group:travel" thumbnail-width="100%" thumbnail-height="100%" title="可点击图片-居中" >}}

    - 全宽图片

        {{< wide-image src="https://i.loli.net/2019/11/05/WCiUr65LuHG29gT.jpg" title="全宽图片" >}}
