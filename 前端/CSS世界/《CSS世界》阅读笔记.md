[toc]

# 《CSS世界》阅读笔记

# 第一章 概述

## 1、流

**自适应布局**：对具有自适应特性一类布局的统称，包含较广范

**流体布局**：较狭义，包含较少

例如"流体布局"往往是具有自适应性的，但是表格布局虽然也能设置为100%自适应，但是表格并不是流体布局

## 2、table自己的世界

虽然"流"影响了整个CSS世界，但是它并不能影响table，table比CSS还要老，在CSS正式诞生之前就有了，所以"流"的特性对table并不适用

# 第二章 流、元素与基本尺寸

> 标签种类众多，但通常分为两类：块级元素、内联元素

## 1、块级元素

块级元素基本特性：**一个水平流上只能单一显示一个元素**，多个块级元素则换行显示

`块级元素`和`display为block的元素`不是一个概念。例如：

`<li>`元素默认的display值是`list-item`

`<table>`元素默认的display值是`table`，但是它们均是“块级元素”