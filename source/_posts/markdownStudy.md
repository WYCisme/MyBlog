---
title: Markdown学习笔记
reward: false
copyright: false
tags:
  - Markdown
categories:
  - 学习
date: 2020-03-21 11:23:10
---

> Markdown是一种可以使用普通文本编辑器编写的[标记语言](https://baike.baidu.com/item/标记语言/5964436)，通过简单的标记语法，它可以使普通文本内容具有一定的格式。
>
> Markdown具有一系列衍生版本，用于扩展Markdown的功能（如表格、脚注、内嵌HTML等等），这些功能原初的Markdown尚不具备，它们能让Markdown转换成更多的格式，例如LaTeX，Docbook。Markdown增强版中比较有名的有Markdown Extra、MultiMarkdown、 Maruku等。这些衍生版本要么基于工具，如Pandoc；要么基于网站，如GitHub和Wikipedia，在语法上基本兼容，但在一些语法和渲染效果上有改动。

### 常用语法

最常见的Markdown格式选项和键盘快捷键:

| 输出后的效果 | Markdown           | 快捷键             |
| :----------- | :----------------- | :----------------- |
| Bold         | `**text**`         | Ctrl/⌘ + B         |
| *Emphasize*  | *text*             | Ctrl/⌘ + I         |
| [Link]()     | `[title](http://)` | Ctrl/⌘ + K         |
| Inline Code  | \`code\`           | Ctrl/⌘ + Shift + K |
| Image        | `![alt](http://)`  | Ctrl/⌘ + Shift + I |
| List         | * item             | Ctrl + L           |
| Blockquote   | > quote            | Ctrl + Q           |
| H1           | # Heading          |                    |
| H2           | ## Heading         | Ctrl/⌘ + H         |
| H3           | ### Heading        | Ctrl/⌘ + H (x2)    |

### 标题

标题能显示出文章的结构。行首插入1-6个 # ，每增加一个 # 表示更深入层次的内容，对应到标题的深度由 1-6 阶。

- H1 :**# Header 1**
- H2 :**## Header 2**
- H3 :**### Header 3**
- H4 :**#### Header 4**
- H5 :**##### Header 5**
- H6 :**###### Header 6**

### 文本样式

（带“*”星号的文本样式，在原版Markdown标准中不存在，但在其大部分衍生标准中被添加）

- 链接 :`[Title](URL)`
- 加粗 :`**Bold**`
- 斜体字 :`*Italics*`
- 高亮 :`==text==`
- 段落 : 段落之间空一行
- 换行符 : 一行结束时输入两个空格
- 列表 :* 添加星号成为一个新的列表项。
- 引用 :*> 引用内容*
- 内嵌代码 : `alert('Hello World');`
- 画水平线 (HR) :--------
- 方框：- [ ] -

### 图片

使用Markdown将图像插入文章，你需要在Markdown编辑器输入 `![]()` 。 这时在预览面板中会自动创建一个图像上传框。你可以从电脑桌面拖放图片(.png, .gif, .jpg)到上传框, 或者点击图片上传框使用标准的图像上传方式。 如果你想通过链接插入网络上已经存在的图片，只要单击图片上传框的左下角的“链接”图标，这时就会呈现图像URL的输入框。想给图片添加一个标题, 你需要做的是将标题文本插图中的方括号，e.g;`![This is a title]()`.

### 脚注

**脚注不存在于标准Markdown中。**

使用这样的占位符号可以将脚注添加到文本中:`[^1]`. 另外，你可以使用“n”而不是数字的`[^n]`所以你可以不必担心使用哪个号码。在您的文章的结尾，你可以如下图所示定义匹配的注脚，URL将变成链接:

```
这里是脚注[^1]
[^1]: 这里是脚注的内容

这里是脚注[^n]
[^n]: 这里是脚注的内容
```

> from: [百度百科](https://baike.baidu.com/item/markdown/3245829?fr=aladdin)

学习ing····