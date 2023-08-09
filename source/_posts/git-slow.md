---
title: Git添加代理
date: 2023-08-09 16:07:30
categories: git
tags: [github, git , proxy]
---
`git`连接`github`会很慢，本文主要记录一些`git`添加代理的一些方式

## 设置全局`git`代理

```sh
git config --global https.proxy http://127.0.0.1:1080

git config --global https.proxy https://127.0.0.1:1080
```

## 取消全局`git`代理

```sh
git config --global --unset http.proxy

git config --global --unset https.proxy
```

## 单独为`Github`设置代理

```sh
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080
```