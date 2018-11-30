---
title: Git clone 错误汇总
tags: [git]
date: 2018-11-10 12:56:35
categories: 
---

# OpenSSLSSL_read: SSL_ERROR_SYSCALL, errno 10054

```cmd
fatal: unable to access 'https://github.com/weechang/moreco-view.git/': OpenSSLSSL_read: SSL_ERROR_SYSCALL, errno 10054
```
 
```cmd
env GIT_SSL_NO_VERIFY=true git clone https://github.com/weechang/moreco-view.git
```
 
 
# RPC failed; curl 18 transfer closed with outstanding read data remaining

```cmd
#Linux:
export GIT_TRACE_PACKET=1
export GIT_TRACE=1
export GIT_CURL_VERBOSE=1

#Windows
set GIT_TRACE_PACKET=1
set GIT_TRACE=1
set GIT_CURL_VERBOSE=1
```

```cmd
pod repo update
pod install
```

```cmd
git config --global http.postBuffer 524288000
```