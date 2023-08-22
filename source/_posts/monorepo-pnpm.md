---
title: monorepo.tools(monorepo深入浅出及其工具)
date: 2023-08-18 14:56:28
categories: [前端工程化]
tags: [工程化, monorepo]
---

> 本文是[monorepo.tools](https://monorepo.tools/#understanding-monorepos)的对照翻译


## 如何理解`Monorepos`

目前，`Monorepos`非常热门，尤其是在`Web`开发人员中。我们创建了这些资源来帮助开发人员了解什么是`monorepos`，它们能带来什么好处，以及可用的工具来使`monorepo`开发令人愉快。有许多优秀的`monorepo`工具，由伟大的团队建立，具有不同的理念。我们尽最大努力客观地代表每个工具，如果我们做错了什么，我们欢迎拉取请求！我们将重点关注的工具有：`Bazel`（由Google）、`Gradle Build Tool`（由Gradle公司）、`Lage`（由微软）、`Lerna`、`Nx`（由Nrwl）、`Pants`（由Pants Build社区）、`Rush`（由微软）和`Turborepo`（由Vercel）。我们选择这些工具是因为它们在Web开发社区中的使用或认可。

## 什么是`Monorepos`？

让我们来定义一下我们和其他人谈论`Monorepos`时的意思。

**`Monorepos`是一个包含多个不同项目、具有明确定义关系的单一存储库**。`Nrwl`认为，在所有已建立的`Monorepos`工具中，这是对单仓库最一致、最准确的陈述。

### 不只是 `Code Colocation`

考虑一个包含多个项目的存储库。我们确实有`Code Colocation`，但如果它们之间没有明确的定义关系，我们不会将其称`Monorepo`。

同样，如果一个存储库包含一个没有分隔和封装的不连续部分的大规模应用程序，它只是一个大型存储库。你可以给它一个花哨的名字，比如`Garganturepo`，但很遗憾，它不是`Monorepo`。

事实上，这样的存储库是极其单一的，而这往往是人们想到`Monorepo`时首先想到的事情。继续阅读，您会发现一个好的`Monorepo`与`Monolith`恰恰相反。

> A good monorepo is the opposite of monolithic! Read more about this and other misconceptions in the article on [“Misconceptions about Monorepos: Monorepo != Monolith”](https://blog.nrwl.io/misconceptions-about-monorepos-monorepo-monolith-df1250d4b03c).

## 为什么是`Monorepos`?(But why?)

> 让我们更深入地了解这个"无底洞"
> Let's go deeper into the rabbit hole.


### Polyrepo

为了便于讨论，我们假设 `Monorepos` 的反义词是`Polyrepo`。`Polyrepo`是当前开发应用程序的标准方法: 针对每个团队、应用程序或项目的`repo`。通常每个回购都有一个构建工件和简单的构建管道。
![Polyrepo](/images/monorepo-pnpm/polyrepo-practice.svg)
这个行业已经转向了多方代理的方式，这有一个重要的原因：**团队自治**。团队希望自己决定使用哪些库、何时部署应用或库，以及谁可以贡献或使用他们的代码。
![Team Autonomy](/images/monorepo-pnpm/spectrum-real-world.svg)
这些都是好事，为什么团队不应该有所不同？因为这种自主权是由隔离提供的，而隔离损害了协作。更具体地说，这些是`Polyrepo`环境的常见缺点：

- 代码共享繁琐

要在多个存储库之间共享代码，您可能需要为共享代码创建一个存储库。现在，您必须设置工具和CI环境，向存储库添加提交者，并设置包发布，以便其他存储库可以依赖它。并且，我们不要着手协调不兼容的第三方库版本在存储库中的不兼容版本。

- 代码重复严重

没有人愿意经历建立共享库的麻烦，所以团队只需在每个库中编写自己的常用服务和组件的实现。这会浪费前期时间，但随着组件和服务的变化，也会增加维护、安全和质量控制的负担。

- 对共享库和使用者产生昂贵的交叉存储更改

考虑一个共享库中的关键错误或突破性变化：开发人员需要设置他们的环境，以便在具有不连续修订历史的多个存储库中应用更改。更不用说对软件包进行版本控制和发布的协调工作了。

- 工具不一致

每个项目都使用自己的命令集来运行测试、构建、提供服务、lint、部署等等。不一致会造成记住项目间使用哪些命令的心理负担。

### Monorepo
在多重代理中工作时，我们可能会陷入非常棘手的境地。但是，`Monorepo`如何帮助解决所有这些问题呢？

- 没有创建新项目的开销

使用现有的CI设置，如果所有使用者都在同一个存储库中，则无需发布版本化包。
- 各项目的原子性提交

每一次提交，所有事情都协同工作。当你在同一个提交中修复所有内容时，没有所谓的破坏性变更。
- 万物一版本

不需要担心由于项目依赖于第三方库的冲突版本而导致的不兼容性。
- 更强的开发自由

获得使用不同工具和技术编写的应用程序的一致构建和测试方法。开发人员可以放心地为其他团队的应用程序做出贡献，并验证他们的更改是安全的。

### Monorepo 的特征
`Monorepo`有很多优点，但要使其发挥作用，您需要有合适的工具。随着工作空间的增长，工具必须帮助你保持快速、可理解和可管理。
| 特性                               | 优点         |
| ------------------------------------ | -------------- |
| Local computation caching            | Fast           |
| Local task orchestration             | Fast           |
| Distributed computation caching      | Fast           |
| Distributed task execution           | Fast           |
| Transparent remote execution         | Fast           |
| Detecting affected projects/packages | Fast           |
| Workspace analysis                   | Understandable |
| Dependency graph visualization       | Understandable |
| Code sharing                         | Manageable     |
| Consistent tooling                   | Manageable     |
| Code generation                      | Manageable     |
| Project constraints and visibility   | Manageable     |

## Monorepo 工具

![Monorepo Tools](/images/monorepo-pnpm/tools.png)