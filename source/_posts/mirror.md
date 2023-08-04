---
title: 前端修改国内源
date: 2023-08-04 16:52:51
tags: [镜像,国内源]
---

## npm 国内源
```sh
npm config set registry https://registry.npmmirror.com
```

## nvm 国内源

- 设置npm_mirror:
```sh
nvm npm_mirror https://npmmirror.com/mirrors/npm/
```
- 设置node_mirror:
```sh
nvm node_mirror https://npmmirror.com/mirrors/node/
```

## electron 国内源

```sh
# https://npmmirror.com/mirrors/electron/
npm config set electron_mirror=https://npmmirror.com/mirrors/electron/
npm config set electron_builder_binaries_mirror=https://npmmirror.com/mirrors/electron-builder-binaries/
```

## 镜像集合

```sh
# https://npmmirror.com/
# 开源镜像: https://npmmirror.com/mirrors/
# Node.js 镜像: https://npmmirror.com/mirrors/node/
# alinode 镜像: https://npmmirror.com/mirrors/alinode/
# ChromeDriver 镜像: https://npmmirror.com/mirrors/chromedriver/
# OperaDriver 镜像: https://npmmirror.com/mirrors/operadriver/
# Selenium 镜像: https://npmmirror.com/mirrors/selenium/
# electron 镜像: https://npmmirror.com/mirrors/electron/
```