---
title: "Markdown手册"
date: 2023-07-28T11:39:58+08:00
categories:
categories:
- 收件箱
- 工作记录
tags:
- 工作
- 记录
keywords:
- 案例
#thumbnailImage: //example.com/image.jpg
---
makrdown 手册记录
<!--more-->
## 勾选框
- [ ]
- [x]

```
- [ ]
- [x]
```
## 换行
结尾两个空格或多个空格。或者```</br>```标签
## 代码块
```
```  ``` 多行代码块，` ` 反引号标记单行。
```
## 文字格式
```
*斜体*  _斜体_
**粗体**
***加粗斜体***
~~删除线~~
```
## 引用内容
```
> 引用内容
```
## 分割线
```
---
```
## 段首缩进
```
&ensp; or &#8194; 表示一个半角空格
&emsp; or &#8195；表示一个全角空格
&emsp;&emsp; 表示两个全角空格
&nbsp; or &#160; 不断行的空白格
```
## 表格
```
|表头|表头|
|---|---|
|值|值|

单元格中的带有竖线：&#124
斜体内容是1个下划线，粗体内容弄个两个下划线
表内换行:<br>
markdown兼容html的合并单元格

```
## 思维导图
```
 \```mermaid
            graph TD

            A[Client] -->|tcp_123| B
            B(Load Balancer)
            B -->|tcp_456| C[Server1]
            B -->|tcp_456| D[Server2]
 \```
```