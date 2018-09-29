---
title: 说一说我所用到的hexo插件
date: 2018-09-29 14:52:20
categories: 开源之路
tags: [Hexo博客, hexo插件]
---

>在我之前的博文中，介绍了如何用hexo搭建一个自己的博客系统。当然只是简单地搭建，还有很多东西需要补全的。今天我就介绍下，hexo常用插件，以此来丰富你的博客功能、改善用户体验。

<!-- more -->

关于如何使用hexo搭建自己的博客系统，感兴趣的同学可以看我之前的文章[六元一个的私有博客系统，了解一下？](https:blog.weechang.xyz//2018/09/19/六元一个的私有博客系统，了解一下？/)。

下面将详细介绍，我的博客中所使用到的一些hexo插件，丰富博客功能、改善用户体验。

# 字数统计插件

```
npm install hexo-wordcount
```

该插件能够做到统计每一篇文章的字数，以及预估阅读时间。但是如果自身的主题不支持的话，还得自己在主题中添加相关代码。如下

```
<div class="article-word-count">
<span>
    <i class="fas fa-file-word"></i>
    共<%= wordcount(post.content) %>字，
</span>
<span>
    <i class="fas fa-clock"></i>
    大约需要<%= min2read(post.content) %>分钟
</span>
</div>
```

添加后就能清楚地看到每篇博文的字数，及大概阅读用时了。

{% asset_img wordcount.png 字数统计效果 %}

# sitemap 生成插件

```
npm install hexo-generator-sitemap
```

该插件能够生成网站地图，供搜索引擎蜘蛛爬取所用。

# 百度sitemap生成插件

```
npm install hexo-generator-baidu-sitemap
```

由于百度蜘蛛的sitemap编写规则与谷歌等有所不同，所以需要针对百度生成一个专门的sitemap，可以使用该插件

# RSS 源生成插件

```
npm install hexo-generator-feed
```

值得注意的是，rss源生成查件不仅仅需要安装，还得配置。需要在你的项目 _config.yml 里面添加配置信息

```
# RSS plugin
plugin:
- hexo-generator-feed
#Feed Atom
feed:
type: atom
path: atom.xml
limit: 20
```

# nofollow 插件

```
npm install hexo-autonofollow
```

nofollow 插件能够给所有非本站的超链接加上 nofollow 标签，nofollow 标签的意义在于 指示搜索引擎不要追踪（即抓取）网页上的带有nofollow属性的任何出站链接，以减少垃圾链接的分散网站权重！

同时，nofollow插件也需要配置才能生效，同样在你的项目 _config.yml 添加配置

```
#nofollow not include
nofollow:
  enable: true
  exclude:
  - blog.unknowns.info
  - yanhaijing.com
  - 友链domain
```

# 百度URL主动提交插件

```
npm install hexo-baidu-url-submit
```

百度url主动提交的意义在于，及时主动地向搜索引擎提交你站的新增文章url，对于搜索引擎的及时收录有一定的帮助。当然如果是一个高质量的老站，而且蜘蛛爬取频次很快的站的话，就当我没说过这句话吧。

需新增配置

```
baidu_url_submit:
  # count表示一次提交几条最新的url
  count: 1
  host: 网站域名
  token: 百度站长主动提交处得token
  path: baidu_urls.txt
```

token具体查看地址的，百度站长平台-用户中心-站点管理-数据引入-链接提交，token为图中token参数后面的值

{% asset_img baiduurl.png  token%}

另外需要，在修改 _config.yml 的deploy配置选项

```
deploy:
  type: baidu_url_submitter
#  type: git
  branch: master
  repo:
    coding: https://git.coding.net/weechang93/weechang93.coding.me.git
    github: https://github.com/weechang/weechang.github.io.git
  bucket: blog.weechang.xyz
```

**其中deploy的type只能同时存在一个，这就需要在部署与提交url之间相互切换了。**

# 静态资源压缩插件

静态资源压缩插件我知道的有两个，一个是全站压缩，一个是可选压缩，先说可选压缩的吧

## 可选压缩

```
npm install hexo-neat
```

需要在 _config.yml 增加配置文件

```
# hexo-neat
# 博文压缩
neat_enable: true
# 压缩html
neat_html:
  enable: true
  exclude:
# 压缩css
neat_css:
  enable: true
  exclude:
  - '**/*.min.css'
# 压缩js
neat_js:
  enable: true
  mangle: true
  output:
  compress:
  exclude:
  - '**/*.min.js'
```

需要注意的是，css 及 js压缩需要排除掉已经压缩了的资源。另外需要压缩html就不能排除markdown的压缩。

## 全站压缩

```
npm install hexo-all-minifier
```

该插件不需要配置，而且不仅能压缩html、css、js 还能压缩图片。所以我选择了这个插件。

还有个问题就是，压缩就是为了节省空间，毕竟博客放的github或者coding。两者节点都不在大陆，所以网速是个大问题，这才是我选择后者的关键原因，后者没有版权注释，前者有版权注释，感觉注释也是一种浪费啊。不是说版权不重要。

**注意，用压缩插件的时候。如果markdown里面有 pre 标签写的代码的话，建议尽快修改。因为这两个压缩插件对于 pre标签里面的java泛型都不是很友好，都会报错。**

OK，我所用到的hexo插件就介绍到这儿，希望对你有所帮助。