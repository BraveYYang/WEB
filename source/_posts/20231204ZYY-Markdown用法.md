---
title: Markdown用法
tag: Markdown
date: 2023-11-30
categories: 路演文书
---

# Markdown用法
- 要写一个好看的文档，格式和各种引用非常重要，Markdown作为开发过程中的记录文件，也是目前应用最广泛的一个格式，因此开篇先分享一下基本用法。
- 这边主要分享几种最常规的使用方法，主要有标题、换行、强调、代码块、分割线、链接、图片、表格这8种非常常用的用法，其余请参照[官方网站](https://markdown.com.cn)
## 1.标题
- 一级标题：`# ZYY 1`
- 二级标题：`## ZYY 2`
- 三级标题：`### ZYY 3`
- 最简单的解释就是，**n个#代表n级标题**，空格之后加上标题就行
## 2.换行
- 句子末尾加上 `<br>` 或者 `两个空格+Enter键`
```
example1：ZYY <br>
example2：ZYY(Space*2+Enter，括号和汉字自动忽视)
```
- **最好用的还是2个空格+Enter键**，但是你看不见它到底有没有换行，所以 `<br>` 这个是比较直观的，但是打字速度就慢了
## 3.强调
- 粗体（Bold）： `**ZYY** 或者 __ZYY__`
- 斜体（Italic）： `*ZYY* 或者 _ZYY_`
- 粗斜体： `***ZYY*** 或者 ___ZYY___`
```
example1：ZZZ **YYY** YYY  或者ZZZ __YYY__ YYY
example2：ZZZ *YYY* YYY  或者ZZZ _YYY_ YYY
example3：ZZZ ***YYY*** YYY  或者ZZZ ___YYY___ YYY
```
- 这个没啥特殊要求，两种方法都可以，**但是最好每种符号的前后都加上空格，不然有些会失灵**
## 4.代码块 and 代码
```
代码： ` + ` 单个反引号做开头，单个反引号做结尾，中间就是特殊的代码区
代码块：``` + ``` 用三个反引号做开头，三个反引号做结尾，中间就是代码块

example1： ZZZ `YYY` YYY
```
- 这个代码块挺好用的，**特别是想要强调说一些事情的时候，可以给他框出来**，我上面的所有例子都是用代码块罗列的，清晰明了，代码块不一定写代码
## 5.分割线
-  `*** 或者 --- 或者 ___`
```
example1： ***
example2： ---
example3： ___
```
- 就会出现一个非常丑的线，**这三根线的类型是一样的，不管用哪个都是一根线**
## 6.链接
- 链接： `[超链接显示名](超链接地址 "超链接title")`
- 直接上网址或者email地址：<链接地址>
```
example1：ZZZ [YYY](https://github.com/BraveYiYang) YYY
example2：ZZZ [YYY](https://github.com/BraveYiYang "Brave") YYY
example3：<https://github.com/BraveYiYang>
```
- 这个还是很好用的，中括号里面的是你可以随便定义，然后把你小括号的链接赋值在你中括号的定义里面，小括号的双引号里面是你鼠标放上去，他会显示出来你的小标签
## 7.图片
- 插入图片： `[![图片alt](图片链接 "图片title")](超链接)`
```
example1：![ZYY](/ZYY/Z.jpg)
example2：![ZYY](/ZYY/Z.jpg "Brave")
example3：[![ZYY](/ZYY/Z.jpg "Brave")](https://github.com/BraveYiYang)
```
- 这个图片的用处是很多的，我们可以随心所欲的插入图片，**图文并茂**
## 8.表格
-  | 表示表格的线，用他来固定列数，几行就是表格的几行，
- --- 来定义标题行，而且每一列的 --- 数量不一样代表每一列的长度也不一样
- :--- 表示左对齐 ---: 表示右对齐 :---: 表示居中对齐
```
example1：
| Syntax      | Description |
| ----------- | ----------- |
| Header      | Title       |
| Paragraph   | Text        |
example2：
| Syntax | Description |
| --- | ----------- |
| Header | Title |
| Paragraph | Text |
example3：
| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |
```
- **表格其实不是很常用，但是也是算作是一个基本的使用方法**
# 总结
- **markdown的使用方法其实不难，而且也很容易入门，只不过没有那么直观，但是大家用习惯之后，就非常容易上手，所以鼓励大家多使用，熟能生巧，多练才会掌握的快！**