# 高效的使用Git管理代码
大家好！😄

感谢大家的时间来阅读此文，如果您对以下内容感兴趣，欢迎关注我的公众号《**叨叨叨的成长记录**》，这里你可以收获以下内容：

1.  专业的IT内容分享
2.  前沿LLM技术和论文分享
3.  个人对行业的思考
4.  投资理财的经验和笔记

_**如果您也对这些感兴趣，欢迎在后台留言，大家多多交流！**_

* * *

_**前序文章**_

*   [面试官系列：Makefile中的高级用法](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FGKJ0mdgWzwwQb4ydolW5Dw "https://mp.weixin.qq.com/s/GKJ0mdgWzwwQb4ydolW5Dw")
*   [面试官系列：你使用Makefile来构建工程么？](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FOMda0V6WE3dJs1qX54edXQ "https://mp.weixin.qq.com/s/OMda0V6WE3dJs1qX54edXQ")
*   [面试官系列：如何理解深度学习中的几种正则化层？](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2F6g-3a46ycMudqus6nr5XOw "https://mp.weixin.qq.com/s/6g-3a46ycMudqus6nr5XOw")

* * *

Git之前如何管理代码？
------------

Git 是在 2005 年由 Linus Torvalds 设计和开发的，主要是为了满足 Linux 内核开发的需要。以下是 Git 诞生的背景和原因：

在 Git 被创建之前，Linux 内核的开发使用的是 BitKeeper，这是一个商业版本控制系统。BitKeeper 提供了一些强大且有效的功能，但由于其商业性质，Linux 内核开发者与 BitKeeper 的开发者之间的关系逐渐恶化。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/1dc7c1e1b15a43e294309cca52468b6a~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6L6b5byD55a-5aWL56yU:q75.awebp?rk3s=f64ab15b&x-expires=1728227544&x-signature=86qVoVWMdgrJ%2FlAdxHh8VuMU%2FM4%3D)

由于 BitKeeper 是一个封闭源码的系统，Linux 开发者在使用过程中面临以下问题：

*   **许可和使用限制**：随着 BitKeeper 的许可政策变更，Linux 内核开发者不能继续免费使用。
*   **开放源代码的需求**：Linux 内核本身是一个开源项目，开发者希望在版本控制工具上也能有同样的开放性。

Linus Torvalds希望构建一个新的版本控制系统，能够满足以下标准：

*   **开放源码**：Git 必须是开放源代码的，允许任何人自由地使用和修改。
*   **强大的性能**：能够高效地处理大规模的项目（如 Linux 内核）。
*   **分布式架构**：支持分布式版本控制，使得每个开发者都能够拥有完整的代码库。
*   **简单易用**：尽可能简化常见的操作。

Git被创造出来
--------

在 2005 年，Linus Torvalds 便开始构建 Git，经过几个月的开发，Git 于同年 4 月首次发布。开发过程中，Torvalds 强调了性能和可用性的关键性，使 Git 的设计和实现迅速获得开发者的认可。

Git 迅速在开源社区内获得普及，特别是在多项目和分布式团队的需要增加后。随后，许多开发者和公司开始使用 Git，并在其基础上构建了许多工具和平台（如 GitHub、GitLab 和 Bitbucket）。

Git基础的使用
--------

以下命令掌握了是可以处理日常工作中60%的操作了，这些是 Git 中一些常用的命令，掌握这些命令可以帮助你更好地管理代码版本和团队协作。对Git基础命令比较熟悉的同学可以快速跳过一下内容。

我们先去看下一个 git repo 中的最为核心的 .git 目录中的内容。

```plain
.git
├── COMMIT_EDITMSG
├── HEAD
├── config
├── description
├── hooks
├── index
├── info
├── logs
├── objects
├── packed-refs
└── refs

```

当然！以下是 `.git` 目录每个文件和子目录的详细说明，帮助你更深入地了解 Git 的内部结构和功能：

*   **COMMIT\_EDITMSG**
    *   **描述**：这个文件用于存储上一次未完成的提交信息。当你执行 `git commit` 时，如果没有填写提交信息，Git 会将你输入的内容（如果有）保存在这里。
*   **HEAD**
    *   **描述**：`HEAD` 文件指向当前检出的分支或提交。它的内容通常是一个引用，指向 `.git/refs/heads/` 下的某个分支，这表示你当前处于哪个分支中。
*   **config**
    *   **描述**：本地 Git 仓库的配置文件，包含特定于该仓库的设置，如远程仓库的 URL、用户信息（用户名和邮箱）等。这些设置会覆盖全局配置。
*   **description**
    *   **描述**：描述仓库的内容，通常在 GitWeb 或其他可视化工具中使用，适用于裸仓库（bare repository），一般开发过程中不太会修改。
*   **hooks**

> 注：钩子脚本文件名通常是以 `.sample` 结尾的示例脚本，需要移除后缀才能激活。

```markdown
- 描述：这个目录包含一系列钩子（hooks），这些钩子是可执行的脚本，Git 在特定事件发生时会自动运行这些脚本。这些事件包括提交代码、合并分支、推送等。
- 常见钩子示例：
    * pre-commit: 在运行 git commit 之前运行，可以用于执行代码检查。
    * post-commit: 在成功提交后运行，用于执行后续操作，比如发送通知。

```

*   **index文件**
    *   **描述**：也称为暂存区（staging area），它是一个二进制文件，保存已暂存的文件的信息。执行 `git add` 的时候，修改的文件信息会被写入这个文件。它用于处理即将提交的更改。
*   **info目录**
    *   **描述**：用于存储一些 Git 信息，通常包括排除文件。
    *   **exclude**
        *   **描述**：类似于 `.gitignore` 文件，用于指定不想追踪的文件。这个文件只影响当前仓库，不会被提交。
*   **logs目录**
    *   **描述**：记录 Git 操作的历史，包括分支的变动。
    *   **HEAD**
        *   **描述**：保存 `HEAD` 指针的历史变动，每次切换分支或操作时都会记录。
    *   **refs**
        *   **描述**：记录所有引用的历史变动，跟踪各个分支和标签的变更。
*   **objects目录**
    *   **描述**：存储 Git 的所有数据对象，包括提交（commit）、树（tree）和文件内容（blob）。这是 Git 的核心数据结构。
    *   **info**
        *   **描述**：包含额外的信息文件，用于 Git 内部命令如 `git fsck` 等。
    *   **pack**
        *   **描述**：用于存储已压缩的 Git 对象，以节省磁盘空间和加快性能。
*   **packed-refs文件**
    *   **描述**：存储已压缩的引用信息，包括分支和标签。使用打包引用可以提高性能，尤其是在拥有大量引用时。
*   **refs目录**
    *   **描述**：存储所有引用（refs），包括分支和标签。
    *   **heads**
        *   **描述**：存储本地分支的引用，目录中的每个文件对应一个本地分支。
    *   **remotes**
        *   **描述**：存储远程分支的引用，包含来自远程仓库的分支信息。
    *   **tags**
        *   **描述**：存储标签（tags）引用的目录，标签用于标识特定的版本。

### 1\. 基本命令

*   **初始化仓库**：

```bash
git init

```

初始化一个新的 Git 仓库。

*   **克隆仓库**：

```bash
git clone <repository-url>

```

克隆一个远程 Git 仓库到本地。

### 2\. 状态与信息

*   **查看状态**：

```bash
git status

```

显示当前工作目录和暂存区的状态。

*   **查看提交历史**：

```bash
git log

```

显示提交历史记录。

### 3\. 文件管理

*   **添加文件到暂存区**：

```bash
git add <file>

```

将指定文件添加到暂存区，准备提交。

*   **添加所有文件到暂存区**：

```bash
git add .

```

将当前目录下的所有变化添加到暂存区。

*   **提交更改**：

```bash
git commit -m "commit message"

```

提交暂存区的更改，并附上提交信息。

### 4\. 分支管理

*   **查看所有分支**：

```bash
git branch

```

列出所有本地分支。

*   **创建新分支**：

```bash
git branch <branch-name>

```

创建一个新的分支。

*   **切换分支**：

```bash
git checkout <branch-name>

```

切换到指定的分支。

*   **创建并切换到新分支**：

```bash
git checkout -b <branch-name>

```

*   **合并分支**：

```bash
git merge <branch-name>

```

将指定分支的更改合并到当前分支。

### 5\. 远程操作

*   **添加远程仓库**：

```bash
git remote add <remote-name> <remote-url>

```

添加一个远程仓库。

*   **查看远程仓库**：

```bash
git remote -v

```

列出所有远程仓库及其 URL。

*   **推送到远程仓库**：

```bash
git push <remote-name> <branch-name>

```

将本地分支推送到远程仓库。

*   **拉取远程更新**：

```bash
git pull <remote-name> <branch-name>

```

从远程仓库拉取更新并合并。

### 6\. 其它

*   **查看差异**：

```bash
git diff

```

查看未暂存的文件的更改。

*   **撤销更改**：

```bash
git checkout -- <file>

```

撤销工作目录中某个文件的改动。

*   **重置暂存区**：

```bash
git reset <file>

```

将文件从暂存区移除，但保留变更在工作目录中。

Git的高级场景化教程
-----------

### 如何规范的Merge代码？

在一个团队协作的 Git 仓库中，规范化代码提交和合并流程对于维护清晰的提交历史和有效的版本管理至关重要。以下是一些推荐的最佳实践，可以帮助团队有效地管理 Git 提交和合并：

#### 1\. 采用分支策略

##### 1.1 功能分支（Feature Branches）

*   **创建功能分支**：每个新功能或修复应在独立的分支上进行开发。分支名称通常包含功能描述，比如 `feature/login-page` 或 `bugfix/issue-123`.

##### 1.2 维护主分支

*   **保护主分支**：通常有一个主分支（如 `main` 或 `master`），只应在完成的功能通过审核后合并到主分支。

#### 2\. 提交规范

##### 2.1 清晰的提交信息

*   **格式**：提交信息应简洁、清晰，遵循一定的格式。例如：

```plain
Type: Short description (max 50 chars)

More detailed description, if necessary.

Related issues: #1234

```

常见类型包括：

*   `feat`: 新特性
*   `fix`: 修复
*   `docs`: 文档改动
*   `style`: 格式（不影响代码的变动）
*   `refactor`: 代码重构
*   `test`: 增加测试
*   `chore`: 其他改动

##### 2.2 小粒度提交

*   **频繁提交**：在开发过程中，应进行频繁的小粒度提交，以便更容易回溯和理解提交的目的。

#### 3\. 代码合并策略

**选择合并方式**：团队需要规定合并时使用的方式，常见的方式包括：

*   **直接合并**（Merge）：用于小的功能和较少冲突的合并。
*   **重新基（Rebase）**：保持提交历史的线性，使历史看起来更干净。

使用命令来完成合并时，可以使用：

```bash
git merge --no-ff feature/my-new-feature

```

或在合并前先进行 rebase：

```bash
git fetch origin
git checkout feature/my-new-feature
git rebase origin/main

```

Git 的 `rebase` 操作是一个非常强大的工具，它被用来重写提交历史，特别是在处理多个开发分支时。下面将通过图示和文字说明，清晰地讲解 `git rebase` 的好处。

##### Rebase 命令

`git rebase` 主要有两种使用场景：

1.  **将一个分支的更改添加到另一个分支上**。
2.  **将多个提交简化为一个提交**，以保持历史整洁。

###### 1\. 提高提交历史的可读性

假设我们有以下的提交历史：

```plain
A---B---C (main)
     \
      D---E (feature)

```

在上述图中，`main` 分支有提交 A, B, 和 C，而 `feature` 分支基于 B 创建了提交 D 和 E。

\=> 使用 `git merge`

如果我们直接使用 `git merge`，合并过程如下：

```bash
git checkout main
git merge feature

```

这将产生一个新的合并提交：

```plain
A---B---C---F (main)
     \     /
      D---E (feature)

```

缺点：

*   历史比较复杂，难以追踪。
*   合并提交 F 可能不含重要信息。

\=> 使用 `git rebase`

而如果我们使用 `git rebase`，过程如下：

```bash
git checkout feature
git rebase main

```

这会将 `D` 和 `E` 提交应用于 `C` 之后，产生新的提交：

```plain
A---B---C---D'---E' (feature)

```

然后你切换回 `main` 分支并合并：

```bash
git checkout main
git merge feature

```

合并后的图如下：

```plain
A---B---C---D'---E' (main, feature)

```

优点：

*   提交历史更线性，易于跟踪。
*   没有合并提交，简化提交历史。

###### 2\. 简化提交历史

另一种适用场景是使用 \`interactive rebase\`（交互变基）来压缩多个提交。例如：

在 `feature` 分支上，假设我们有：

```plain
A---B---C (main)
     \
      D---E---F (feature)

```

如果我们希望将 `D`, `E`, `F` 合并为一个提交，并将信息整合，可以执行：

```bash
git rebase -i HEAD~3

```

在交互界面中，选择 `squash` 或 `fixup` 将 D, E, F 合并为一个提交。

最终结果：

```plain
A---B---C---G (main)
          \
           (feature)

```

这里 `G` 是合并后的新提交。

优点：

*   提交历史更干净，减少了无关的小提交。
*   让功能的实现更易于理解。

#### 4\. 代码同步

*   **保持更新**：开发人员应定期将主分支的更改合并到他们的功能分支，以避免重大合并冲突。
*   **及时解决冲突**：发现冲突后，应该及时解决，确保主分支和功能分支始终是可用的。

### 如何同步社区的Git Repo？

#### 场景描述

当你发现有一个功能或者算法在社区上已经有了开源的项目（Github Repo），你想持续的更社区的功能迭代保持一致，但是你又需要在这个Repo之上进行一些二次开发，这时你要如何管理你的项目呢？

1.  **从 GitHub 中获取最新的改动**。
2.  **对本地代码进行二次开发**。
3.  **将更改推送到公司内部的 Git 仓库**。

#### 操作步骤

##### 1\. 克隆 GitHub 仓库

首先，从 GitHub 上克隆项目到你的本地机器：

```bash
git clone https://github.com/username/repo.git
cd repo

```

**解释**：

*   `git clone <repository-url>`：将远程 GitHub 仓库克隆到本地。
*   `cd repo`：进入克隆下来的项目目录。

##### 2\. 添加公司内部的 Git 仓库作为远程

接下来，为了将更改推送到公司内部的 Git 仓库，你需要添加这个仓库作为远程：

```bash
git remote add internal <internal-repo-url>

```

**解释**：

*   `git remote add internal <internal-repo-url>`：将公司内部的 Git 仓库添加为远程，命名为 `internal`。

##### 3\. 定期同步 GitHub 的最新改动

在开始开发之前，确保从 GitHub 获取最新的改动。切换到主分支并拉取最新更新：

```bash
git checkout main
git pull origin main

```

**解释**：

*   `git checkout main`：切换到主分支。
*   `git pull origin main`：从 GitHub 的 `main` 分支拉取最新的更改到本地。

##### 4\. 创建新的开发分支

为了进行二次开发，建议你创建一个新分支：

```bash
git checkout -b feature/my-new-feature

```

**解释**：

*   `git checkout -b feature/my-new-feature`：创建并切换到新的开发分支。

##### 5\. 进行二次开发

在新分支上进行你的代码更改，完成后你可以将更改添加到暂存区并提交：

```bash
git add .
git commit -m "Implement my new feature"

```

**解释**：

*   `git add .`：将所有更改添加到暂存区。
*   `git commit -m "message"`：提交更改并附上描述信息。

##### 6\. 保持与 GitHub 的同步

在开发过程中，继续定期将 GitHub 的最新更改合并到你的开发分支。首先，切换回主分支并拉取最新更改，然后合并到你的开发分支：

```bash
git checkout main
git pull origin main
git checkout feature/my-new-feature
git merge main

```

**解释**：

*   这些命令是前面提到的步骤，用于获取最新的 GitHub 更改并将其合并到开发分支。

##### 7\. 处理合并冲突（如果有）

如果合并过程中发生冲突，你需要手动解决这些冲突：

1.  打开有冲突的文件，手动解决冲突。
2.  然后，将解决后的文件添加到暂存区并提交：

```bash
git add <resolved-file>
git commit -m "Resolve merge conflicts"

```

##### 8\. 推送到公司内部 Git 仓库

完成开发后，你可以将更改推送到公司内部的 Git 仓库。确保在推送前切换到你的开发分支：

```bash
git push internal feature/my-new-feature

```

**解释**：

*   `git push internal <branch-name>`：将你的开发分支推送到公司内部的 Git 仓库。

### 如何迁移某个commit？

`git cherry-pick` 命令是一个非常有用的工具，主要用于将特定的提交（commit）从一个分支应用到另一个分支，而不需要合并整个分支。

*   **冲突处理**：在使用 `cherry-pick` 时，如果所选择的提交与目标分支存在冲突，你将需要手动解决这些冲突。
*   **提交历史**：`cherry-pick` 的结果会创建一个新的提交，如果你需要保持原始提交的作者信息和时间戳，可以使用 `-x` 选项。

下面介绍下 `cherry-pick` 的主要使用场景及其目的：

#### 1\. 修复特定错误

当在一个功能分支上发现了 bug 并进行了修复，你想将这个修复迅速应用到主分支或其他分支，而不想合并整个功能分支。使用 \`cherry-pick\` 可以方便地将这个修复提交应用到其他分支。

```bash
git checkout main
git cherry-pick <commit-hash>

```

#### 2\. 选择特定的提交

在开发过程中，可能会有多个提交，部分是你想要的，但其他的没有必要合并。例如，在功能开发分支上有多个提交，其中只有几个提交涉及到某个功能或修复。使用 \`cherry-pick\` 可以选择性地将这部分提交应用到主分支。

#### 3\. 简化提交历史

在某些情况下，你可能希望更新某个分支的内容，而不想增加复杂的合并提交。通过 \`cherry-pick\`，你可以把需要的提交直接应用到目标分支，从而保持提交历史的清晰与简洁。

#### 4\. 应用一个已经被删除的提交

有时你可能在处理时删除了某个重要的提交，\`cherry-pick\` 可以帮助你恢复这个提交。如果该提交仍可在其他分支上找到，只需复制其提交哈希并执行 \`cherry-pick\`。

#### 5\. 合并多个特性

在多个功能分支上并行工作时，某些特性可能需要彼此共享。通过对特定提交使用 \`cherry-pick\`，可以将一个分支的特性应用到另一个分支，而不必进行合并。

### 如何修改commit的引用？

*   使用 `git reset --hard` 会清除未保存的更改，因此在执行该命令前务必确认没有未提交的更改。
*   `git reflog` 的历史记录会随着时间推移而消失，因此及时使用是关键。
*   在恢复分支或提交时，注意与团队成员的协作，确保不会引起不必要的冲突。

#### 1\. 恢复误删除的分支

假设你误删了一个名为 `feature-branch` 的分支：

```bash
git branch -d feature-branch

```

1.  查看 reflog 以找到之前的提交：

```bash
git reflog

```

2.  查找与删除分支相关的提交，记下最后一次提交的哈希值（如 `abc1234`）。
3.  恢复分支：

```bash
git checkout -b feature-branch abc1234

```

#### 2\. 撤销错误的提交

假设你最近进行了一个错误的提交，想要回退到前一个状态。

1.  查看 reflog 以找到你想要回退到的提交：

```bash
git reflog

```

2.  找到正确的提交哈希值（如 `def5678`）。
3.  回退到该提交：

```bash
git reset --hard def5678

```

#### 3\. 找回丢失的提交

假设你进行了一次 `git reset` 操作，导致某些提交丢失。

1.  查看 reflog 以找到丢失的提交：

```bash
git reflog

```

2.  记下丢失提交的哈希值（如 `ghi9012`）。
3.  使用该哈希值创建新的分支以保存该提交：

```bash
git checkout -b recovered-branch ghi9012

```

#### 4\. 查看和理解操作历史

你想回顾最近的操作。

1.  查看 reflog：

```bash
git reflog

```

2.  使用输出信息了解每个操作的上下文，比如提交、合并、重置等。

#### 5\. 解决合并冲突后的状态

在合并某个分支时遇到冲突，想要回到合并之前的状态。

1.  使用 reflog 查看合并前的状态：

```bash
git reflog

```

2.  找到合并之前的提交哈希（如 `jkl3456`）。
3.  回退到合并前的状态：

```bash
git reset --hard jkl3456

```