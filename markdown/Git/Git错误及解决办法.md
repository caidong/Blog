---
title: Git常见错误
date: 2021-08-10 23:12:43
tags:
- Git
- 笔记
categories: 
- Git
---
###  1. git clone 报证书错误

```
Cloning into 'ChickenFarm'...
fatal: unable to access 'https://github.com/caidongHui/ChickenFarm.git/': server certificate verification failed. CAfile: none CRLfile: none
```
解决办法：

```
git config --global http.sslVerify false
```