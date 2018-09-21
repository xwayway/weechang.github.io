---
title: IDEA、WebStorm最新永久激活方式
date: 2018-09-21 13:22:40
categories: 日常记录
tags: [日常记录, IDEA永久激活, WebStorm永久激活]
---

>今天早上一大早打开IDEA发现激活已过期，遂开始寻找激活码。但是一直不成功，后来终于找到一种比较靠谱的激活方式。在此记录下来，以备不时之需。

目前网上现有的激活方式大概有这么三种

# 激活码

这种方式一般是给出一段激活码，然后有些还需要改host。目前这种方式，很多激活码都不能使用了，就算能使用可能到了某一天你会发现他用不了了。所以不太推荐这种方式。


# 服务器

这种方式一般是填写一个服务器地址就行了，但是现在很多服务器都被官方封了。没被封的也在被封的路上。

# 自建服务器

这种方式比较靠谱，自己搭建一个认证服务器。但是对于没有服务器的穷人来说，也是一个巨大的挑战。所以不是特别推荐。

接下来就推荐一种比较靠谱的方式。

# 插件激活

这种方式呢目前而言比较靠谱，由于插件在本地，不存在官方封域名及IP的说法，但是不排除被后期修复的问题。至少目前而言是没问题的，亲自验证过IDEA2017.3.5与2018.2是没有任何问题的。

下面将具体介绍这种方式的操作方法。

## 下载插件

首先下载插件jar包 [http://idea.lanyus.com/jar/JetbrainsCrack-3.1-release-enc.jar](http://idea.lanyus.com/jar/JetbrainsCrack-3.1-release-enc.jar)

## 安装插件

然后将插件移动到 IDEA 安装目录的bin下。如我的目录 D:\idea\IntelliJ IDEA 2018.2.4\bin

## 修改配置

修改 IDEA 安装bin目录下的 idea64.exe.vmoptions 或者 idea.exe.vmoptions 具体修改哪一个看自己操作系统是64 位还是32位。64位修改 idea64.exe.vmoptions ， 32位修改 idea.exe.vmoptions。

{% asset_img 配置文件.png 配置文件 %}

然后在文件末尾添加上

<pre>

-javaagent:D:\idea\IntelliJ IDEA 2018.2.4\bin\JetbrainsCrack-3.1-release-enc.jar

</pre>

其中 D:\idea\IntelliJ IDEA 2018.2.4\bin\JetbrainsCrack-3.1-release-enc.jar 为插件的安装路径和文件名。注意，目录和文件名一定要一直，不然IDEA不能启动

{% asset_img 修改配置.png 修改配置 %}

## 重启IDEA、填写CODE

重启IDEA，选择Activation Code填写激活码

<pre>
    ThisCrackLicenseId-{
    "licenseId":"ThisCrackLicenseId",
    "licenseeName":"你想填的用户名",
    "assigneeName":"",
    "assigneeEmail":"你想填的邮箱",
    "licenseRestriction":"For This Crack, Only Test! Please support genuine!!!",
    "checkConcurrentUse":false,
    "products":[
    {"code":"II","paidUpTo":"2099-12-31"},
    {"code":"DM","paidUpTo":"2099-12-31"},
    {"code":"AC","paidUpTo":"2099-12-31"},
    {"code":"RS0","paidUpTo":"2099-12-31"},
    {"code":"WS","paidUpTo":"2099-12-31"},
    {"code":"DPN","paidUpTo":"2099-12-31"},
    {"code":"RC","paidUpTo":"2099-12-31"},
    {"code":"PS","paidUpTo":"2099-12-31"},
    {"code":"DC","paidUpTo":"2099-12-31"},
    {"code":"RM","paidUpTo":"2099-12-31"},
    {"code":"CL","paidUpTo":"2099-12-31"},
    {"code":"PC","paidUpTo":"2099-12-31"}
    ],
    "hash":"2911276/0",
    "gracePeriodDays":7,
    "autoProlongated":false}
</pre>

{% asset_img 激活码.png 激活码 %}


# 激活

OK，至此为止，你的IDEA 就激活成功，又可以继续浪了。

** Web Storm 激活步骤与此相同 **