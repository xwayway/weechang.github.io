---
title: 使用GitBook制作电子书
date: 2018-09-04 13:35:54
categories: 开源之路
tags: [手把手教学]
---

>由于最近想要将AxonFrameWork的官方英文文档翻译成中文。为了方便自己及广大爱好者浏览，决定制作成在线电子书的形式。于是想到了GitBook，在此将自己的GitBook制作之路记录下来，留给有需要的朋友。

<!-- more -->

# 简介

由于文笔有限，简介就直接扒度娘的吧。
GitBook 是一个基于 Node.js 的命令行工具，可使用 Github/Git 和 Markdown 来制作精美的电子书，GitBook 并非关于 Git 的教程。
GitBook支持输出多种文档格式：
* 静态站点：GitBook默认输出该种格式，生成的静态站点可直接托管搭载Github Pages服务上；
* PDF：需要安装gitbook-pdf依赖；
* eBook：需要安装ebook-convert；
* 单HTML网页：支持将内容输出为单页的HTML，不过一般用在将电子书格式转换为PDF或eBook的中间过程；
* JSON：一般用于电子书的调试或元数据提取。

# 环境依赖
* Node.js（推荐使用v4.0.0及以上版本）
    由于GitBook是基于Node.js的命令行工具，所以Node.js的环境依赖是必须的。具体的Node.js搭建环境，请各位自行度娘，不再啰嗦。
* Windows，Linux，Unix 或 Mac OS X（各大主流操作系统均可）

# 安装
GitBook最简单的安装方法便是通过npm命安装，在命令行输入以下命令即可安装

<pre>
 npm install gitbook-cli -g
</pre>

安装完成后可用过，以下命令查看GitBook版本

<pre>
 gitbook -V
</pre>

><b>注意</b>
此处V一定是大写，至少windows环境下是，其他环境还未证实

# 创建你的第一本书

以下将具体介如何创建并部署属于你的电子书
    
## 初始化

<pre>
 gitbook init {directory}
</pre>

{directory} 表示具体目录，通过该命令，你可以在指定目录下初始化你的GitBook。

## 构建

<pre>
 gitbook build
</pre>

通过运行该命令，会在项目的目录下生成一个 _book 目录，里面的内容为静态站点的资源文件，你可结合git管理工具等，将该文件推送到GitHub、码云、Coding等git平台上，通过gitPages的方式展示你的电子书。后面将会对该方式做具体介绍。

## 本地预览

<pre>
 gitbook server
</pre>

通过运行该命令，将在本地启动一个静态服务器，通过本地浏览器，你可预览你的gitbook内容。默认端口为4000 。本地访问地址为 http://localhost:4000/

# GitBook目录

## 项目结构

GitBook的目录结构很简单。在 SUMMARY （即 SUMMARY.md 文件）中列出的所有 Markdown 文件都将被转换为 HTML。

一个基本的 GitBook 电子书结构通常如下：

<pre>
.
├── book.json
├── README.md
├── SUMMARY.md
├── chapter-1/
|   ├── README.md
|   └── coentent.md
└── chapter-2/
    ├── README.md
    └── coentent.md
</pre>

特殊文件介绍：

| 文件 | 介绍 |
| ------ | ------ | 
| book.json | 配置数据 (optional) |
| README.md	| 电子书的前言或简介 (required) |
| SUMMARY.md | 电子书目录 (optional) |
| GLOSSARY.md |	词汇/注释术语列表 (optional) |

## 静态资源目录

静态文件是在 SUMMARY.md 中未列出的文件。除非被忽略，否则所有静态文件都将复制到输出路径。

## 忽略文件与文件夹

GitBook将读取 .gitignore，.bookignore 和 .ignore 文件，以获取要过滤的文件和文件夹。这些文件中的格式遵循 .gitignore 的规则：

<pre>
    ## 忽略以下文件
    test.md
    a.md

    ## 忽略该文件夹下所有文件
    test/*
    target/*   
</pre>

# 配置

## 常见配置

| 文件 | 介绍 |
| ------ | ------ | 
|root |	包含所有图书文件的根文件夹的路径，除了 book.json |
|structure |	指定自述文件，摘要，词汇表等的路径，参考 Structure paragraph. |
|title |	您的书名，默认值是从 README 中提取出来的。在 GitBook.com 上，这个字段是预填的。 |
|description |	您的书籍的描述，默认值是从 README 中提取出来的。在 GitBook.com 上，这个字段是预填的。
|author |	作者名。在GitBook.com上，这个字段是预填的。 |
|isbn |	国际标准书号 ISBN |
|language |	本书的语言类型 —— ISO code 。默认值是 en |
|direction |	文本阅读顺序。可以是 rtl （从右向左）或 ltr （从左向右），默认值依赖于 language 的值。 |
|gitbook |	应该使用的GitBook版本。使用 SemVer 规范，并接受类似于 “> = 3.0.0” 的条件。 |


# GitBook 部署

## 托管到gitbook.com




