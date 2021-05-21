---
title: 敏捷开发模型中关于Git的思考
top: true
cover: true
toc: true
mathjax: true
date: 2019-08-03 16:31:12
password:
summary: 以 Git 为例，介绍在敏捷开发模型中如何进行版本管理。
tags: 
  - 软件工程
  - Git
categories: 软件工程
---

一个成功的 DevOps 实践，需要以一个成熟优秀的 “版本管理” 为核心的 。当下最流行的版本控制平台非 [Git](https://git-scm.com/) 莫属！这篇文章将以它为例，介绍在 DevOps 中如何进行版本管理，并使项目易于维护。

Git 之所以如此流行，以我片面的认识是由于 Git 相较于传统的版本控制平台，它的分支操作不会生成代码的物理拷贝，而是以指针的形式指向当前版本（又称为“快照”），因此十分便捷、易于使用。然而，使用方便的副作用则是如果管理不佳，将会创造出一个枝节四处开放的项目，难以协调和维护。

一个比较合理，并适应敏捷开发模型的分支管理策略是 [Vincent Driessen](http://nvie.com/) 提出的 [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/) （以下简称 “该策略”），我从中收获很多。它可以使得版本库的演进保持简洁，主干清晰，各个分支各司其职、井井有条。理论上，该策略对所有的版本管理系统都适用。当然这篇文章的主角是 Git ，我将会以 Git 举例来实现这个模型。

![git 分支模型](https://www.helloimg.com/images/2021/05/21/BfBdFn.jpg)

（图片来源：[A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/) ）

如上图所示，该策略主要包含三类分支：主分支（master）、开发分支（develop）和 辅助性分支（feature、release、hotfix）。

## 1. 主分支 Master

 版本库需要有且仅有一个主分支，这个分支通常是 master 。master 分支会在版本库初始化之后自动创建，所有提供给用户的正式版本，都应该是在该分支上。如下图：

![master 分支](https://www.helloimg.com/images/2021/05/21/BfBQt5.jpg)

这里的每一个节点对应一次提交操作，命令如下：

```bash
git add <some file>
git commit -m "some commit message"
git push
```

假设我们正在开发一个 Web 应用，我们通过 CI/CD 工具去监听我们的版本库，如果开发人员提交了代码，将会自动触发构建，并将自动将代码发布到预先配置好的环境中去运行，这样用户就可以直接访问到我们的 Web 页面。

现在新手程序员小陈，通过上述的命令向版本库提交了新的代码，此时我们的版本库中只有一个 master 分支，提交的代码可能只是在小陈的环境中测试通过了，甚至小陈可能没有测试直接进行了提交。由于没有进行完整的测试，小陈提交的代码有可能导致我们的 Web 应用被恶意利用，甚至无法访问。

因此，我们需要另外的一个分支，用于日常开发。

## 2. 开发分支 develop

我们把 master 分支用来发布重大版本，日常开发应该在另一条分支上完成。通常我们把开发用的分支叫做 develop。如下图所示：

![develop 分支](https://www.helloimg.com/images/2021/05/21/BfBvnR.jpg)

这里我们需要一些额外的操作来创建一个新的分支：

```bash
git checkout -b develop master
```

有了 develop 分支之后，我们就可以在 CI/CD 平台中新建一个任务流去监听它，并将它发布到测试环境，进行功能测试和安全测试。至此，程序员小陈就可以放心地提交代码，而不用担心影响用户体验了！

在测试环境完成功能测试和安全测试通过之后，我们就可以考虑将 develop 分支合并（merge）到 master 分支上：

```bash
# 首先我们需要先切换到 master 分支
git checkout master
# 将改动合并到当前分支 （即，将 develop 分支合并到 master 分支）
git merge --no-ff develop
```

这里稍微解释一下 ``--no—ff`` 参数的作用（ff => fast forward）。我们之前提到过，Git 的合并操作实际上是用类似指针的方式，将指针指向了当前的快照。而默认情况下，如果我们直接使用不带参数的 ``merge``  去合并 develop 分支，将会执行 “快进式合并” 把 master 分支的指针直接指向 develop 分支，如下图：

![快进式合并](https://www.helloimg.com/images/2021/05/21/BfBNBA.jpg)

这和我们预想的不同。使用 ``--no--ff`` 参数，可以使 master 分支中新增一个节点，并合并 develop 中的改动，就如同我们一开始所想的那样，两条分支并行前进。

## 3. 辅助性分支

其实在大多数小规模项目的开发中，使用上述的两个分支已经足够了，一条用于发布，一条用与日常开发。

但是针对规模较大，或者有特殊要求的项目，我们需要一些辅助性分支来保证我们主干分支的健壮性。

辅助性分支主要有以下三类：

- 功能分支（feature）
- （预）发布分支（release）
- 热修复分支（hotfix）

这些分支是为了某些需求而临时生成的，使用完之后就应该删除，因此辅助性分支也被称为临时性分支。代码库常设的分支应该是开发分支和主分支。

### 3.1 功能分支

功能分支是为了开发特定功能而从 develop 分支上新建的，待开发完成后再合并会 develop 分支。

例如，最近我们的 Web 应用需要添加支付模块，项目技术负责人认为小陈最近状态不错，便将这个任务分配给了小陈。小陈需要带领一个小团队来攻克这个问题，他先从 develop 分支新建了一个名为 “feature-PayModule” 的分支：

```bash
git checkout -b feature-PayModule develop 
```

经过他们的努力，终于完成了该功能，他们需要把这个模块合并回 develop 让 CI/CD 平台自动将改动发布到测试环境进行测试：

```bash
# 切换分支
git checkout develop
# 合并分支
git merge --no-ff feature-PayModule
# 删除 feature 分支
git branch -d feature-PayModule
# 提交代码
git push origin develop
```

### 3.2 发布分支

发布分支也叫作预发布分支（pre-release），通常它是在版本正式发布前的一个预发布测试版，它允许我们在正式上线前可以做一定的修改。

预发布分支是从 develop 分支上分出，最终将会合并到 develop 和 master 分支上。市面上大多数软件都有一个夜间预览版（nightly），基本上都是通过 release 版本发布的。

从 develop 分支创建 release 分支的要求是所有计划发布的 feature 已经合并到了 develop 分支上，而不计划发布的分支需要等到创建 release 分支之后才能合并到 develop 分支。创建 release 分支的时候需要为 release 分支设定一个版本号，因为将 release 分支将会合并到 master，而 master 分支的每一次 commit 就相当于一个版本的发布，所以将带版本号 release 分支合并到 master 将会使版本更迭更加清晰。

现在，假设我们的 Web 应用现在希望新增 Preview Version，具有内测资格的特殊用户群体会帮助我们测试即将发布的新功能。小陈需要这样做：

```bash
# 创建一个 release 分支
git checkout -b release-0.4 develop

# 经过内测用户测试，并修改问题后，合并到 master 分支
git checkout master
git merge --no-ff release-0.4

# 对合并生成的新节点，做一个标签
git tag -a v0.4

# 再合并到 develop 分支
git checkout develop
git merge --no-ff release-0.4

# 最后删除 release 分支
git branch -d release-0.4
```

### 3.3 热修复分支

软件正式发布后，难免会出现一些Bug，比如可能被恶意利用的漏洞，或者老旧设备、系统等适配出现问题。这就需要我们临时创建一个分支，来进行 Bug 修复。这个分支就被称为 hotfix 分支。

hotfix 分支是从 master 分支上分出来的，最后将会合并到 master 分支和 develop 分支。

例如，在我们发布 v4.0 版本之后，有用户通过页面上的 FeedBack 反馈在 IE 7 上的样式布局变得很奇怪，并难以使用。项目经理收到这个反馈后，任命小陈去针对 IE 7 做一个版本适配，而其他研发人员继续完成 develop 分支上的开发。

小陈接到该任务后，使用如下方式进行 hotfix 分支上的开发：

```bash
# 创建一个 hotfix 分支
git checkout -b hotfix-0.4.1 master

# 修复结束后，合并到 master 分支
git checkout master
git merge --no-ff hotfix-0.4.1
git tag -a v0.4.1

# 再合并到 develop 分支
git checkout develop
git merge --no-ff hotfix-0.4.1

# 最后删除 hotfix 分支
git branch -d hotfix-0.4.1
```

## 小结

使用该模型可以大大增加开发的效率，并且让每一次 commit、merge 的目的变得更明确。我们可以通过一些工具、平台的辅助来更好地完成这些工作。下面我推荐几款效率工具：

|     工具作用     |                           工具名称                           |                           推荐原因                           |
| :--------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 代码版本控制平台 |             [Gitlab](https://about.gitlab.com/)              |          开源、支持本地私有化部署、丰富的插件库支持          |
|   持续集成平台   |     [Jenkins]([https://jenkins.io](https://jenkins.io/))     |            开源、多种部署方式，中文支持，插件丰富            |
|   问题追踪平台   |       [JIRA](https://www.atlassian.com/software/jira)        |   基于 Issue 驱动的项目管理非常适合敏捷，安全性、扩展性强    |
| 白盒代码审计平台 | [CxSAST](https://www.checkmarx.com/products/static-application-security-testing/) + [SonarQube](https://www.sonarqube.org/) |  两款工具语言支持丰富，集成性能优秀，并能与上述平台无缝集成  |
| 灰盒代码审计平台 | [CxIAST](https://www.checkmarx.com/products/iast-interactive-application-security-testing/) | 不会延长 DevOps 时间，可以与 CxSAST 结果交叉验证结果汇总，并且能与上述平台良好集成。 |
