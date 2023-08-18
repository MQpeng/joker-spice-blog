---
title: PNPM是什么以及为什么前端开发者应该尝试使用它
date: 2023-08-18 14:28:55
categories: [前端基础]
tags: [pnpm]
---

> 本文是[《What is PNPM & Why You Should Try It As a Frontend Developer》](https://javascript.plainenglish.io/what-is-pnpm-why-you-should-try-it-as-a-frontend-developer-69a3a7b34f5b)的对照翻译

了解 pnpm——`npm/yarn`的替代方案，了解其优点、依赖管理和用法。

前置疑问：为什么使用 PNPM，它有哪些学习点？

## 什么是 pnpm?

正如[官网](https://pnpm.io/zh/)所说，**快速的，节省磁盘空间的包管理工具**

## 优势

1. 快速的

以下是几个工具的比较：

可以看出，在大多数情况下，pnpm 作为黄色部分，包安装速度明显优于`npm/yarn`，速度会比`npm/yarn`快 2 倍。
| action | cache | lockfile | node_modules | npm | pnpm | Yarn | Yarn PnP |
| ------- | ----- | -------- | ------------ | ----- | ----- | ----- | -------- |
| install | | | | 37.7s | 19.3s | 22.1s | 20.2s |
| install | ✔ | ✔ | ✔ | 2s | 1.6s | 695ms | n/a |
| install | ✔ | ✔ | | 8.9s | 5.1s | 8.8s | 668ms |
| install | ✔ | | | 13.7s | 8.6s | 22.8s | 15.2s |
| install | | ✔ | | 14.2s | 16.8s | 8.9s | 670ms |
| install | ✔ | | ✔ | 2.4s | 3.9s | 16s | n/a |
| install | | ✔ | ✔ | 2s | 1.6s | 681ms | n/a |
| install | | | ✔ | 2.3s | 15.2s | 16.6s | n/a |
| update | n/a | n/a | n/a | 8.9s | 8s | 8.7s | 16.9s |

![Benchmarks](/images/pnpm/alotta-files.svg)

总体而言，`pnpm`的包安装速度仍然明显优于 yarn。

2. 节省磁盘空间

在内部管理中，`pnpm`使用内容可寻址的文件系统来存储磁盘上的所有文件。这个文件系统的优点是：

相同的包不会重复安装。当使用`npm/yarn`时，如果有 1000 个项目依赖于`lodash`，那么`lodash`可能会被安装 1000 次，并且这部分代码会在磁盘上被写入 1000 个地方。然而，当使用`pnpm`时，它只会安装一次，并且只在磁盘上写入一个地方。如果你再次使用它，你将会直接使用硬链接。

即使对于不同版本的包，`pnpm`也能够大大重用之前版本的代码。例如，如果`lodash`有 1000 个文件，并且在更新版本之后又增加了一个文件，那么磁盘将不会重写 1001 个文件，而是保留原来的 1000 个文件的硬链接，只写入新的一个文件。

3. 安全性高

在使用`npm/yarn`之前，由于 node_module 的扁平结构，如果`A`依赖于`B`，`B`依赖于`C`，则`A`可以直接使用`C`，但问题是`A`没有声明`C`作为依赖项。因此，会出现这种非法访问的情况。然而，`pnpm`有非常大的“大脑洞”，并创建了一套依赖项管理方法，很好地解决了这个问题，并确保了安全性。

## `pnpm`依赖管理

回顾一下`npm/yarn`为什么想要扁平化`node_modules`，难道不是因为相同的依赖项会被复制多次，路径也变得太长了吗？

如果你不复制文件，比如通过链接来替代呢？

首先，我将介绍链接，即软链接和硬链接。这是操作系统提供的机制。硬链接是同一文件的多个引用，而软链接则是创建一个新的文件，该文件的内容指向另一个路径。当然，这两种链接的使用方式是相似的。

如果你不复制文件，只需将`npm`包的内容保存在全局仓库中，然后将其他地方都链接起来。

这样，就不会有多个拷贝导致的浪费磁盘空间的问题，也不会有路径过长的问题。因为路径过长的主要限制是目录层次结构不能太深，现在每个位置的目录都是链接，而不是同一个目录，所以没有长度限制。

没错，`pnpm`就是通过这个想法实现的。

然后删除 node_modules，然后使用`pnpm`重新安装它，并执行`pnpm install`。

你会发现它打印出这句话：

```sh
pnpm install
```

![output](/images/pnpm/1_IxAcMI5AHQqvpKu0J_2RaQ.webp)
展开`.pnpm`文件夹可以看待
![.pnpm](/images/pnpm/1_Cewnyamzt3myXhsMe8PiiA.webp)
所有依赖项都布局在这里，通过全局存储硬连接，然后通过软链接组织包到包的依赖项。
例如，在.pnpm 下的 express，这些是软链接
![.pnpm](/images/pnpm/1_HiWI8PDjCOC3tXpIxs1UCA.webp)
也就是说，所有依赖项都从全局存储硬链接到 node_modules/.pnpm，然后通过软链接相互依赖。

官方提供了一个示意图，您可以一起查看以理解它：
![.pnpm](/images/pnpm/1_KiXOhjIsgvXEKoWkWMQ8eA.webp)

## 使用

### pnpm install

```sh
# Install axios
pnpm install axios
# Install axios and add axios to devDependencies
pnpm install axios -D
# Install axios and add axios to dependencies
pnpm install axios -S
```

## 结语

**感谢谢阅读**，我期待着您的关注和阅读更多高质量的文章。
