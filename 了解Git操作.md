# 了解Git操作
### 0.前言

Git仓库中的提交记录保存的是你目录下的所有的文件快照，就像把这些文件粘贴复制了一样，但是要比粘贴复制更加优雅。Git希望提交记录尽可能保持轻量，因此在每次提交的时候，它不会盲目的去复制你的整个目录。条件允许的情况下，它会将当前版本与仓库中的上一个版本进行对比，并把所有的差异打包到一起作为一个提交记录。

Git还保留了每次的提交记录，这是为什么大多数节点都有parent节点的原因。同样对于项目组来说维护Git提交历史对大家都有好处。

我相信这篇文章能够让你学会Git 90%+的基础操作和了解到一些进阶命令。

多图警告！！！ ⚠️ ⚠️ ⚠️

### 1\. Git基础篇

#### 1.1 Git Commit

Git 提交（Git Commit）是 Git 版本控制中的一个基本操作，它会创建项目当前状态的快照，保存对仓库中文件所做的更改。它记录了修改、添加和删除操作，使开发者能够跟踪项目历史并进行有效的协作。

当我们在本地执行命令git commit时，会生成一条新的记录，并且最新记录会指向上一个记录即parent节点，同时分支（branch）指针将会自动移动。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/7719f91033e54671b0a3830783762cd5~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=CfbGtbbIaRugP6dlx5W8DQlavlU%3D)

#### 1.2 Git Branch

Git 分支（Git Branch）是指向 Git 仓库中特定提交（commit）的一个轻量级指针。它允许开发者同时在不同的功能或项目版本上工作，而不会干扰主代码库。

Git的分支也是非常轻量的，它们仅仅是简单的指向某一个提交记录而已，所以即使创建再多的分支也不会造成存储和内存上的压力。

多用分支能够将不同的工作逻辑分解到不同的分支，这要比维护单个特别臃肿的分支容易多。

如何创建和使用分支呢？

*   可以通过命令git branch 创建一个新的分支。
*   可以通过命令git checkout 切换分支。

例：git checkout foo 切换到分支foo上面，然后就可以在foo分支commit新的代码。

下面我们模拟：

1.  创建新分支bugFix。git branch bugFix
2.  切换到分支bugFix上。git checkout bugFix
3.  在bugFix分支上commit一次代码。git commit

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/e75744990d43486c90e85c81296c78e8~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=gv4H6pNFWgX%2F0JyD4MTcrxNe2wo%3D)

好的经过上面三步，我们成功创建了分支bugFix，并在bugFix分支完成了一次提交。

> main\*表示我们当前所处的分支是main分支。

#### 1.3 Git Merge

Git 合并（Git Merge）是 Git 版本控制中的一个命令，用于将不同分支的更改合并到一个单一的分支中。它会创建一个新的提交（commit），将这些更改整合在一起，通常用于将功能分支合并回主开发分支。

好的，看到这里相信你已经学会了如何去提交代码，如果创建分支并使用它们🧐。接下来该学一学如何将两个分支合并到一起了，也就说我们在某一个分支上面开发新功能或者修复bug，开发完成后回归主线，也就是将次分支代码合并到主分支上🤔。先学学第一种方法git merge。

git merge操作会产生一个新的特殊提交记录，它有两个parent节点，**大概就是将这两个parent节点本身以及它们的祖先都包含进来。** 

假设我们有两个分支，一个主分支main，一个bug修复分支bugFix。

我们已经将bug修复的代码提交，即c2记录，现在需要将bug修复的代码合并到主分支上面，开展后续的开发工作。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/0d3207086ee44c209694d26695ecf541~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=Rm2kL28lxlSoKdoyRGDxYkYR1Mc%3D)

*   我们在main分支将bugFix分支的新提交合并过来，执行命令git merge bugFix。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/4665af05faf042df9d8a3057e7ca2ca0~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=yX7OiMsi6BmYUkyiczJvej96eqM%3D)

可以看到创建了一个新的提交记录c4，它有两个parent节点，且这两个节点在不同的分支，还记得我们前面说的\*\*合并大概就是将这两个parent节点本身以及它们的祖先都包含进来。\*\*假如从 `main` 开始沿着箭头向上看，在到达起点的路上会经过所有的提交记录。这意味着 `main` 包含了对代码库的所有修改。

思考一下如果我们在bugFix分支将main分支的代码合并过来会怎么样？

*   我们在bugFix分支将main分支的代码合并过来，执行命令git merge main。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/8e46ca2730b94cfc87e51224ac384d1c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=mDGa6n8oudd%2BOieW5ckOLFUgoG0%3D)

可以看到git只是将bugFix的指针，指向了c4提交，这是因为main分支继承自bugFix，Git什么都不用做直接改变bugFix分支的指向即可。

#### 1.4 Git Rebase

Git Rebase是 Git 版本控制中的一个命令，允许开发者将一系列提交（commit）移动或合并到一个新的基础提交上，从而创建一个更加线性的项目历史。它是合并的另一种替代方式，可以使提交时间线更加清晰、简洁。

🤓这个命令就是我们要介绍的第二个合并命令了，它是取一系列的提交记录，复制它们，然后依次在另外一个地方放下去。它相对于merge的优势在于可以创建更加线性的提交历史。

我们现在有两个分支主分支main，bug修复分支bugFix，我们现在要将bugFix上的代码合并回main分支上面。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/f2f0e1a05b2b44b0817b6bd0b55de538~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=NFtCZQOuNPRE%2FeDuNDPBa97L1dw%3D)

*   执行命令git rebase main

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/2e08e2774fb744ad82bac4d55ed1f7de~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=myzRYTARQycwLpi%2Fzc%2FDzEYGZ0c%3D)

我们可以看到git是将c2提交记录，复制了一份，放到了c3记录下。注意c2记录并没有被删除，它依然是存在的c2‘是我们复制到main分支上的副本。

现在还有一个问题是main分支没有更新到最新提交处，我们来更新main分支吧，首先切换到main分支上面，执行命令git rebase bugFix

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/bfed88bed74b44b3923cb577be64be20~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=s22JvS%2FpCQgBpOb39TGskT04Nbg%3D)

到这里Git的基础就讲完了，什么？这么简单？ 🤨，后面开始进入Git的一些进阶操作和概念了。

### 2\. Git高级篇

#### 2.1 分离HEAD

在学习git高级操作技巧前，我们要学会如何在Git提交树上移动。

首先解释一下HEAD的概念，HEAD是一个对当前分支的符号引用，也就是一个指针，它指向的是你正在进行操作的提交记录上。**HEAD总是指向当前分支上面最近的一次提交记录，它通常情况下是指向分支名称的，如main。大多数修改提交树的Git命令都是从改变HEAD指向开始的。** 

我们可以通过git checkout 命令来进行分离HEAD操作。

*   指向命令git checkout c1

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/245a1a6b709746919177ad1f75cb65c8~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=ITBxM3uSsF8ptDt%2F%2B56qQYIi058%3D)

> 实际这些命令并不是真的在查看 HEAD 指向，这里只是用图示进行抽象，方便理解。
> 
> 如果想查看 HEAD 指向，可以通过 `cat .git/HEAD` 查看， 如果 HEAD 指向的是一个引用，还可以用 `git symbolic-ref HEAD` 查看它的指向。

当然也可以将HEAD与分支进行分离。

*   执行命令git checkout c3

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/67ce9396019741608f4bd698a16f40d1~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=bgBlTaC8Qnw4D2JgNPpnL1vllnI%3D)

实际使用git checkout 命令是，hash并不会向图示的一样简单c1，c2等。如果需要获取提交记录的hash至，可以通过git log命令来查看。实际的hash是基于SHA-1，共有40位，我们可以只取它的前几位，也可以进行git checkout的操作。

通过哈希值指定提交记录很不方便（要查找对应记录的hash值），所以 Git 引入了相对引用。

使用相对引用的话，你就可以从一个易于记忆的地方（比如 `bugFix` 分支或 `HEAD`）开始计算。

相对引用非常给力，两个简单的用法：

*   使用 `^` 向上移动 1 个提交记录
*   使用 `~<num>` 向上移动多个提交记录，如 `~3`，如果后面没有带数字默认移动单位是1。

如命令git checkout main^表示，相对于main分支向上移动一个单位，也就是执行当前分支最新提交记录的parent节点。git checkout main~2，指的是相对于main分支向上移动两个单位。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/d4c79328b03042259abe8b2336d2e679~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=GvqZL9f%2FLXTSzNUpxW%2BSgeTciuw%3D)

#### 2.2 强制修改分支位置

我使用相对引用最多的就是移动分支。可以直接使用 `-f` 选项让分支指向另一个提交。例如:

`git branch -f main HEAD~2`

上面的命令会将 main 分支强制指向 HEAD 的第 2 级 parent 提交。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/e0c3dc6b2d0247339d7c0cf64295c464~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=9mlGaZAISNqBoBEt0uNCffPQ6NU%3D)

#### 2.3 撤销更改

主要有两种方法用来撤销变更 —— 一是 `git reset`，还有就是 `git revert`。接下来咱们逐个进行说明。

*   Git Reset

git reset是通过将分支记录回退几个记录来实现撤销更改，这里回退的记录是指的本地的commit并非远程仓库的。如下图所示，相当于删除了本地最新的一次提交。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/b71ff549c8274b9aa39af8b60c6a247b~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=nW5Elrdi5Pare4r%2BMgvwKPPj2TU%3D)

*   Git Revert

如果你的记录已经push到远程仓库，此时你想撤销你本次的更改，并将你的撤销分享给项目的其他成员，那么要使用git revert命令了。

git revert它的原理是复制你要删除节点的parent节点，然后更改当前分支指针指向这个新复制的节点。当然你要撤销的记录仍然是存在的，这里相当于再次提交了一次你都上一次提交记录。下面图示操作表示撤销当前提交。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/3cc1f755f2e14e858408646ac31d28bb~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=Og0CCgf%2B%2FrnbVxWu3zBXtlU%2FuoE%3D)

这样就可以push当远程仓库与项目组的其他成员共享。

* * *

到这里差不多已经学会Git 90%的内容，这些操作应对日常开发戳戳有余。下面开始一些进阶操作，这些操作你可能用不到，但学习它们不会错的。

* * *

### 3\. 自由修改提交树

这里主要说的是提交记录的整理。

#### 3.1 Git Cherry-pick

它的作用是将指定的提交记录复制到当前HEAD下。

git cherry-pick …参数位提交记录的hash值。

git cherry-pick c2 c4 这条命令会把c2，c3记录复制到当前head下（从左到右依次复制），也就是main下面。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/ac434ec64e564dad8fc0c56cf6566e80~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=q8SOQnOeebTrwT0a1yZKy9WiHY4%3D)

#### 3.2 Git Rebase -i

git rebase -i HEAD~4（列出当前HEAD指向的前四条记录，包括当前指向记录，作为备选记录进行操作）执行这条命令，Git 会打开一个 UI 界面并列出将要被复制到目标分支的备选提交记录，它还会显示每个提交记录的哈希值和提交说明，提交说明有助于你理解这个提交进行了哪些更改。

大概是这样样子：

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/97750389002442b18a4410b5077a4c69~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=WCHjpVbKN3aOUiiMmdWFtN9h6wY%3D)

执行该命令时，会将相关操作全部列出供参考。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/ba4cdfc439e142bda0bd704fee85106b~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=AYlbftzEOl0vd6tPRUJs3i6gBmw%3D)

这里其实是一个文本编辑器vim，操作和vim是相同的。在这里您可以整理（删除，修改，排序）指定的这几条提交记录。编辑完成确认后git保存更改。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/d5eff7b68fd24218b3d32c91f8f6f815~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=JRVvcQOmDNuLAGyFzdLyA7QuH%2F4%3D)

### 4\. 标签与定位

分支很容易人为移动，并且当有新提交的时候它也会移动，分支很容易被改变，而且有些分支甚至是临时的。那有没有可以永久指向某一个提交记录的表示呢？比如我们发布了某个版本或者修复了什么重大的bug，需要一个比分支更好可以永久指向这个提交记录的方法呢？

git tag的作用就是如此，它可以永久的将某个提交进行命名，打上标记，然后就可以向分支一样引用了。

例如我们给c1提交记录打一个v1的tag进行标记。git tag v1 c1

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/3e9053d713cb44cf99392337959f7038~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=6HcZ1bVYtl9f%2BfAIgkAp8tYHnlU%3D)

*   git describe

git describe 命令能够帮助你在提交树上多次移动后找到方向，它的语法是git describe ，ref是可以被git识别的提交记录引用，如果没有指定该参数的话，默认是HEAD，也就是你当前的位置。

他会输出tag\_numCommits\_g hash，`tag` 表示的是离 `ref` 最近的标签， `numCommits` 是表示这个 `ref` 与 `tag` 相差有多少个提交记录， `hash` 表示的是你所给定的 `ref` 所表示的提交记录哈希值的前几位。 当 `ref` 提交记录上有某个标签时，则只输出标签名称。

### 5\. 远程操作

我们通过git clone 命令将远程仓库的代码下载到本地，链接的形势可以是https或者ssh（使用 `http://` 或 `git://` 协议）。**Git 远程仓库的操作实际可以归纳为两点：向远程仓库传输数据以及从远程仓库获取数据。** 

介绍一下远程分支：

远程分支反映了远程仓库(在你上次和它通信时)的**状态**。远程分支有一个命名规范 —— 它们的格式是:

*   `<remote name>/<branch name>`

大多数开发人员会将他的远程仓库命名为origin，这是因为当你用git clone时git已经帮你把远程仓库的名称设置为了origin。

当我们切换到远程分支上时，比如git checkout origin main，那么会自动变成分离head状态，此时在进行提交origin main分支也不会更新，这是因为只有远程仓库中相应分支更新时origin main才会更新。画个图理解一下这个过程：

1.git checkout origin main

2.git commit

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/a0fb1fd39cdc4108b12b7d4bcc1bd4e6~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=pB%2B6Y4419EZZ5%2F6E5jCPE%2Fs1vEU%3D)

#### 5.0 远程分支

git好像知道main分支是与origin/main分支关联的。在进行pull操作的时候，提交记录会先被下载到origin/main分支上面，之后在合并到本地的mian分支上面，push操作的时候我们将工作的mian分支代码推送到远程仓库的main分支上面。

这个关联关系是由分支的remote tracking属性决定的。main被设定为跟踪origin/main，这意味这mian分支指定了拉取和合并以及推送的对象。

当你克隆这个仓库的时候，git就已经帮你将这个关联设定好了，git会为远程仓库的每一个分支创建一个本地分支如origin/main，然后创建一个跟踪远程仓库的本地分支如main。

一般在克隆完成后你会看到： `local branch "main" set to track remote branch "o/main"`

当然你也可以自己指定这个属性，举个例子

git checkout -b notMain origin/main 这样我们就可以创建一个本地分支notMain来追踪远程分支origin/main。

也可以使用git branch -u origin/main notMain结果是一样的。

#### 5.1 Git Fetch

git fetch命令的作用是从远程仓库下载本地仓库中缺少的记录并更新远程分支指针（没有更新本地分支指针），画个图示理解一下这句话：

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/7e83461ca83c4f62b481f4e07a566a78~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgTXVTaGFuWXU=:q75.awebp?rk3s=f64ab15b&x-expires=1728370930&x-signature=Rmrli%2FOllMxg7Fn0qyr89HU%2FxNM%3D)

然后使用git rebase origin main就能将本地代码更新与远程分支代码一致了， 当然也可以git merge origin main。

`git fetch` 实际上将本地仓库中的远程分支更新成了远程仓库相应分支最新的状态。注意：`git fetch` 并不会改变你本地仓库的状态。它不会更新你的 `main` 分支，也不会修改你磁盘上的文件。git fetch操作可以理解为单纯的下载操做。

> 至于git fetch后是用git rebase还是git merge，这点就是仁者见仁智者见智了，有的偏爱git rebase，因为git rebase能够让你拥有一个有着线性提交历史的git提交树，但是rebase操做修改了提交树，比如说c1提交记录可以被rebase到c3提交记录后，这看起来好像c3是在c1之前进行的工作，实际是相反的，有些人喜欢保留清晰的操作历史，他们就喜欢使用git merge。如果你喜欢干净的提交树，那么我推荐你使用rebase命令，如果你喜欢完整的提交历史，那么我推荐你使用merge命令。

*   fetch参数
    
    fetch参数与push参数是类似的只是方向不同罢了，fetch是从远程到本地，push是反过来的。
    
    1.  git fetch
        
        git fetch origin foo，意思是到远程仓库的分支foo上，然后获取所有本地不存在的提交，下载到本地的origin/foo分支上面。
        
    2.  git fetch origin :
        
        git fetch origin foo:foo 这里表示的意思和git fetch origin foo是一样的。这里的source参数表示的是远程仓库的git可识别位置。destination参数表示的是本地仓库git可以识别的位置。
        
        当然source参数可以不指定，git fetch origin :foo这样会在本地创建一个foo分支。
        

#### 5.3 Git Pull

实际上从远程仓库抓取更新最新代码，然后合并更新本地分支，这个操做非常常用，因此git提供两一个命令一次完成这两个操做，就是git pull命令。

git pull命令实际上是git fetch和git merge操做的合并。当然也有命令git pull —rebase命令表示git fetch和git rebase命令的组合。

*   pull的一些参数
    1.  git pull
        
        git pull origin main，这里实际上是git fetch origin main，git merge origin/main。
        
    2.  git pull origin bar:bugFix，表示拉取远程分支bar的代码，合并到本地分支bugFix上面，相当于git fetch origin bar:bugFix，git merge bugFix。
        

#### 5.4 Git Push

我们已经知道如何从远程仓库拉取代码，那么我们如何将自己的代码与项目组的其他成员进行共享呢？我们可以使用git push命令将自己的代码推送到远程仓库，与其他人进行共享。

git push命令负责将你的更改上传至指定的仓库，并在远程仓库上面合并你的新提交记录，

*   push的一些参数
    1.  git push
        
        git push origin main这个表示切换到本地的main分支找到所有的commit，在到远程仓库origin中找到main分支，然后将远程仓库没有的提交记录都添加进去，添加完成后告诉我结果。通过place参数告诉git，提交记录来自哪，推送到哪里。
        
        > 注意，我们通过place参数切换了分支。
        
    2.  git push origin :
        
        source参数表示的十一个git能够识别的位置，比如一个分支side或者一个分离HEAD，HEAD~。
        
        git push origin side~:main，这里表示将本地分支side的上一个提交记录和之前未被提包含的记录推送到远程main分支上面。如果你要推送的分支不存在那么就会自动帮你创建对应的分支。
        
        git push origin main:newB，这个命令会在远程仓库将newB分支创建，同时也会创建本地分支origin newB。然后你就可以使用git branch -u origin/newB newB将本地分支与远程分支进行remote tracking了。
        

### 6\. 复杂场景

ok，学了这么多命令和原理，是时候将这些命令灵活运用去解决一些可能会遇到的场景。比如最简单的代码冲突你应该怎么做？想一想 🤔，check it out！