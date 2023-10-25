---
title: Monorepo及其相关工具
date: 2023-08-18 14:56:28
categories: [前端工程化]
tags: [工程化, monorepo]
---

> Monorepo已经成为一种常见的项目管理策略，它允许开发者在一个集中的代码库中管理和维护多个项目。
本次技术分享将介绍Monorepo的概念，通过常见的前端项目管理痛点，讲解monorepo的解决方案，以及如何使用Lerna和Nx等工具实现Monorepo管理。

### 项目代码仓库管理策略的演进

前端从最初的三剑客（HTML、CSS、JavaScript）到现在的各种框架、库、工具的百花齐放，前端项目的代码量也越来越多。在项目代码量不断增大的情况下，项目代码仓库的管理策略也在不断演进。目前比较流行的代码仓库管理策略有：

1. `Monolith` 策略： 流行于三剑客时代，前后端代码统一放在一个代码仓库中，前端代码和后端代码共同维护，形成一个巨石单体应用
2. `Multi-repo` 策略：前后端分离时代，组件库、模块库、工具库等公共依赖会单独建立一个代码仓库，前端项目也会按照业务进行拆分成多个代码仓库，通过包管理工具(`npm`,`yarn`,`pnpm`)来实现项目对公共库的依赖
3. `Mono-repo` 策略：微前端、微模块时代，多个业务项目最终会组合形成单一应用，公共库和项目库统一管理，公共库的升级消除了滞后性，项目库之间存在的公共代码公共模块可以及时的拆离出来，具有清晰的依赖关系，同时项目的代码风格、版本管理、提交规范、CI/CD、测试、部署等也可以得到更好的统一管理

![evolution](/images/monorepo-pnpm/evolution.png)

### 实战讲解

实现monorepo的方案有很多，这里我们介绍两种比较流行的方案：

![evolution](/images/monorepo-pnpm/project.png)

#### `pnpm`实现`Monorepo`

`pnpm`特性里面有`Workspace`，我们可以通过这个特性来实现`Monorepo`。

1. 创建一个`monorepo`项目，初始化`pnpm`
```shell
mkdir monorepo
cd monorepo
pnpm init
```
2. 创建`pnpm-workspace.yaml`文件
```yaml
# ./pnpm-workspace.yaml
packages:
  # 项目目录
  - 'packages/*'
  # 公共库目录
  - 'components/**'
  - 'api/**'
  - 'modules/**'
  # 排除测试目录中的文件
  - '!**/test/**'
```
3. 创建公共库和项目目录，并通过`pnpm`初始化
```bash
mkdir packages components api modules
# 初始化公共库目录
cd components
pnpm init
# 初始化项目目录
mkdir packages/project-{a,b,c}
cd packages/project-a
pnpm init
```
4. 修改`package.json`文件中的`name`字段，为了方便管理，可以统一加上`@monorepo`前缀
```json
// package.json
{
    "name": "@monorepo/modules",
}
```
5. 构建依赖关系
```bash
# 公共库之间的依赖
cd modules
pnpm add @monorepo/api
# 项目依赖
# project-{a,b}依赖公共库
cd packages/project-a
pnpm add @monorepo/api @monorepo/modules @monorepo/components
cd packages/project-b
pnpm add @monorepo/api @monorepo/modules @monorepo/components
# project-c依赖project-a和project-b
cd packages/project-c
pnpm add @monorepo/project-a @monorepo/project-b
```
```json
// 安装完后的packages/project-a/package.json
{
  "dependencies": {
    "@monorepo/api": "workspace:^",
    "@monorepo/components": "workspace:^",
    "@monorepo/modules": "workspace:^"
  }
}
```
6. 实例测试
```ts
// api/user.ts
export type User = { name: string; age: number; sex: 'male' | 'female' }

export const fetchUserList = () => {
    return new Promise<User[]>((resolve, reject) => {
        setTimeout(() => {
            resolve(
                new Array(10).fill(null).map((_, i) => ({
                    name: 'name' + i,
                    age: i,
                    sex: (i & 1) === 1 ? 'male' : 'female',
                }))
            );
        }, 1000);
    });
};

// api/index.ts
export * from './user'

// package/project-a setup
import { ref } from 'vue'
import { fetchUserList, type User } from '@monorepo/api'
const user = ref<User[]>([])
fetchUserList().then((list)=>{
  user.value = list
})
```
7. 项目构建部署
```bash
# package/project-a
pnpm run build
# 构建后进入dist目录，模拟服务启动
cd dist
# 本地启动服务
live-server
```

我们已经通过`pnpm`的`Workspace`实现了`Monorepo`。
同时我们仍旧面临使用体验的问题，每次构建项目都需要在项目中执行构建命令(当然也可以通过脚本自动实现)，这对于开发、测试、部署、CI/CD等都难以管理
针对上面的问题，我们可以使用`Lerna`工具，统一管理`script`指令

#### `Lerna`工具使用

`lerna`可以很好的结合`pnpm`，利用各自的优势更好的实现项目管理

1. 安装`Lerna`
```bash
pnpm add -w -D lerna
# 初始化
pnpm lerna init
pnpm lerna changed --all
# 输出项目中的packages
pnpm lerna list
```
2. 执行后会生成`lerna.json`文件
```json
{
  "$schema": "node_modules/lerna/schemas/lerna-schema.json",
  "version": "0.0.0",
  "npmClient": "pnpm"
}
```
3. 统一管理`script`命令
```bash
# 这时候可以在根目录构建project-a
pnpm lerna run --scope project-a build
# pnpm lerna run --scope project-a dev
```

目前我们已经实现了`script`命令中心化管理。

#### `Nx`工具使用

`Nx`是一个由前`Google`员工开发的构建系统，它利用了`Google`内部工具所使用的许多技术。我们可以利用`Nx`来解决以下问题：
1. 代码生成
2. 以`task`的形式管理脚本
3. 管理、缓存、分发`task`
4. Nx Console —— VS Code 插件


1. 安装`Nx`
```bash
pnpm add -w -D nx@latest
```
2. 展示项目依赖关系图
```bash
pnpm nx graph
```
![graph](/images/monorepo-pnpm/graph.png)
3. 代码生成
```bash
# 安装Nx插件
pnpm add -w -D @nx/plugin@latest
# 创建项目插件
pnpm nx g @nx/plugin:plugin my-plugin
# 生成generator
pnpm nx generate @nx/plugin:generator my-generator --project=my-plugin
# 在需要代码生成的项目里面执行
pnpm add @monorepo/my-plugin
# 通过模板生成代码
pnpm nx generate @monorepo/my-plugin:my-generator button
```