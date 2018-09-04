---
title: MySql中的find_in_set、in、like的区别
date: 2018-08-23 10:33:26
categories: 日常记录
tags: [日常记录, MySql]
---

>最近需要做项目，遇到一个需要查询表中的某一列，其数据结构为多个id间用“,”隔开的字符串。需要判断是否包含某一个id，最开始想的是用like查询，但是转念一想，like查询会存在查询id为1的时候，11、12这样的数据也会被查询出来，后来开始寻找其他方法，找到了find_in_set(str, Strs)函数。然后就对MySql中字符串查询的函数一起做一个总结。

<!-- more -->

## in
 <b>in相当于多个or条件查询</b>
 
 例如：
 
 select * from user where id in(1,2,3)

  
## NOT IN

## LIKE

## SUBSTR()

## FIND_IN_SET()