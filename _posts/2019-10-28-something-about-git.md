---
layout: post
title:  "Git 二三事"
categories: git
---

先来聊聊 Git 的开发流程，一般来说 Git Flow 应该有以下几个分支：

- release 测试和预发布
- master 主分支
- feature 功能开发
- hotfix 热修复

不同项目分支应该都差不多，我所在的游戏项目里的分支如下：

- master 功能开发
- stable 热修复

开发新版本以及前端（Unity）Bug 修复，就从 master 创建分支，代码写完后合并到 master。

后端 Bug 修复以及 Admin Tool（游戏后台管理）就从 stable 创建分支，代码写完后合并到 stable。

另外这两个分支的后端有各自对应的测试服务器（使用 Hudson 自动部署），编写完代码之后，就由测试人员进行测试。

新版本的开发流程一般是：

1. Hotfix（stable 分支）
    1. 根据线上玩家反馈的 Bug（已经确认存在且能复现）以及运营的需求（游戏规则或者 Admin Tool 的修改）确定开发内容。
    2. 代码编写完毕且通过测试人员测试。
    3. 后端发邮件给运维（附上 MD5，Commit Tag, OTA Tag 以及 Change Logs 等）。
    4. 测试人员发送确认邮件。
    5. 运维回复邮件。

2. 新版本开发（master 分支）
    1. 策划和运营定下新版本的游戏内容。
    2. 后端讨论设计以及分工。
    3. 根据前端需求，后端写 API Wiki。
    4. 编写程序。
    5. 测试人员测试。
    6. 修改测出来的 Bug。
    7. 提包，发版。

线上 Bug 反馈流程：

1. 玩家提供 Bug 截图以及详细信息给客服。
2. 客服通过 JIRA 将 Bug 提给测试。
3. 测试复现后将 JIRA 转给程序。
4. 程序修复后更新 JIRA 状态。

其他：

- 程序代码合并时，需要将合并请求分配给另一个人进行 Code Review。
- Git 上不能使用 reset 只能使用 revert 进行回滚。
- 若是不想在远端留下 merge commit 的话可以使用 git pull --rebase（这种情况频繁出现在别人 push 了代码，然后你 push 的时候没有拉最新的代码）。

还有一些关于 Git 的奇奇怪怪的技巧，但是实际开发中也用到了很多：

- 如果有想忽略的文件（还未 git add）但不想加到 .gitignore 后 push 到远端，可以在 .git/info/exclude 中添加文件。之所以要这么做是因为项目中的某些依赖实在太旧，自己的开发环境使用了新库，但是不能修改远端指定依赖的文件。
- 如果有想忽略的文件（已经 git add）但不想提交到远端：
    ```
    # Ignore
    git update-index --assume-unchanged <file>
    # Undo
    git update-index --no-assume-unchanged <file>
    # List ignored files
    git ls-files -v | grep '^h'
    ```
- 如果某几次提交对应的 Email 以及 Name 设置错误，可以参照 [GitHub Help](https://help.github.com/en/articles/changing-author-info)（虽然我们用的是 GitLab）。
