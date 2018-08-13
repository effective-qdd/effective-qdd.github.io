---
layout: post
title: markdown 语法示例
date: 2018-8-13
categories: markdowm
tags: markdowm
excerpt: 列出了常用的markdown语法，以作参考。
mathjax: true
---

# Markdown 语法示例

> Markdown 是一种轻量级标记语言，它允许人们使用易读易写的纯文本格式编写文档，然后转换成格式丰富的HTML页面。    —— [维基百科](https://zh.wikipedia.org/wiki/Markdown)

正如您在阅读的这份文档，它使用简单的符号标识不同的标题，将某些文字标记为 **粗体** 或者 *斜体* 或者 ***斜体加粗*** 或者 ~~删除的~~，创建一个[链接](http://www.example.com)或一个脚注[^demo]。

创建TOC (Table of Content):
[TOC]

## 列表

创建**无序列表** ：

* Item1
* Item2
* Item3

或者**有序列表** ：

1. Item1
2. Item2
3. Item3

或者**嵌套的列表** ：

* Item1
  * Item1-1
  * Item1-2
* Item2
  * Item2-1
  * Item2-2

你也可以

1. Item1
    * Item1-1
    * Item1-2
2. Item2
    * Item2-1
    * Item2-2

## 代码块

`int i= 0;`

``` c++
int main()
{
    return 0;
}
```

## LaTeX 公式

可以创建行内公式，例如 $\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$。或者块级公式：

$$ x = \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$

## 表格

| Item      |    Value | Qty  |
| :-------- | --------:| :--: |
| Computer  | 1600 USD |  5   |
| Phone     |   12 USD |  12  |
| Pipe      |    1 USD | 234  |

## 流程图 （github暂不支持）

```flow
st=>start: Start
e=>end
op=>operation: My Operation
cond=>condition: Yes or No?

st->op->cond
cond(yes)->e
cond(no)->op
```

## 时序图 （github暂不支持）

```sequence
Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob-->Alice: I am good thanks!
```

> **提示：**想了解更多，请查看**流程图**[语法][1]以及**时序图**[语法][2]。

## 复选框

* [x] 已完成事项
* [ ] 待办事项1
* [ ] 待办事项2

[^demo]: 这是一个示例脚注。

[1]:http://flowchart.js.org/
[2]:https://bramp.github.io/js-sequence-diagrams/